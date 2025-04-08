# salon.sql
Salon Appointment Scheduler

salon.sh

#! /bin/bash

# Main menu function
main_menu() {
  echo -e "\n~~~~~ MY SALON ~~~~~\n"
  echo "Welcome to My Salon, how can I help you?"
  echo ""
  
  # Display services
  SERVICES=$(psql --username=freecodecamp --dbname=salon -t -c "SELECT service_id, name FROM services ORDER BY service_id;")
  echo "$SERVICES" | while read SERVICE_ID BAR NAME
  do
    echo "$SERVICE_ID) $NAME"
  done
  
  # Get service selection
  read SERVICE_ID_SELECTED
  
  # Check if service exists
  SERVICE_EXISTS=$(psql --username=freecodecamp --dbname=salon -t -c "SELECT COUNT(*) FROM services WHERE service_id = $SERVICE_ID_SELECTED;")
  if [[ $SERVICE_EXISTS -eq 0 ]]; then
    echo -e "\nI could not find that service. What would you like today?"
    SERVICES=$(psql --username=freecodecamp --dbname=salon -t -c "SELECT service_id, name FROM services ORDER BY service_id;")
    echo "$SERVICES" | while read SERVICE_ID BAR NAME
    do
      echo "$SERVICE_ID) $NAME"
    done
    read SERVICE_ID_SELECTED
    SERVICE_EXISTS=$(psql --username=freecodecamp --dbname=salon -t -c "SELECT COUNT(*) FROM services WHERE service_id = $SERVICE_ID_SELECTED;")
    if [[ $SERVICE_EXISTS -eq 0 ]]; then
      exit
    fi
  fi
  
  # Get service name for later use
  SERVICE_NAME=$(psql --username=freecodecamp --dbname=salon -t -c "SELECT name FROM services WHERE service_id = $SERVICE_ID_SELECTED;" | xargs)
  
  # Get customer phone
  echo -e "\nWhat's your phone number?"
  read CUSTOMER_PHONE
  
  # Check if customer exists
  CUSTOMER_ID=$(psql --username=freecodecamp --dbname=salon -t -c "SELECT customer_id FROM customers WHERE phone = '$CUSTOMER_PHONE';" | xargs)
  
  if [[ -z $CUSTOMER_ID ]]; then
    # New customer - get name
    echo -e "\nI don't have a record for that phone number, what's your name?"
    read CUSTOMER_NAME
    
    # Insert new customer
    psql --username=freecodecamp --dbname=salon -c "INSERT INTO customers (phone, name) VALUES ('$CUSTOMER_PHONE', '$CUSTOMER_NAME');"
    CUSTOMER_ID=$(psql --username=freecodecamp --dbname=salon -t -c "SELECT customer_id FROM customers WHERE phone = '$CUSTOMER_PHONE';" | xargs)
  else
    # Existing customer - get name from database
    CUSTOMER_NAME=$(psql --username=freecodecamp --dbname=salon -t -c "SELECT name FROM customers WHERE phone = '$CUSTOMER_PHONE';" | xargs)
  fi
  
  # Get appointment time
  echo -e "\nWhat time would you like your $SERVICE_NAME, $CUSTOMER_NAME?"
  read SERVICE_TIME
  
  # Insert appointment
  psql --username=freecodecamp --dbname=salon -c "INSERT INTO appointments (customer_id, service_id, time) VALUES ($CUSTOMER_ID, $SERVICE_ID_SELECTED, '$SERVICE_TIME');"
  
  # Output confirmation
  echo -e "\nI have put you down for a $SERVICE_NAME at $SERVICE_TIME, $CUSTOMER_NAME."
}

# Database setup (run this separately first)
: '
psql --username=freecodecamp --dbname=postgres -c "CREATE DATABASE salon;"
psql --username=freecodecamp --dbname=salon << EOF
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    phone VARCHAR(15) UNIQUE,
    name VARCHAR(50)
);
CREATE TABLE services (
    service_id SERIAL PRIMARY KEY,
    name VARCHAR(50)
);
CREATE TABLE appointments (
    appointment_id SERIAL PRIMARY KEY,
    customer_id INT REFERENCES customers(customer_id),
    service_id INT REFERENCES services(service_id),
    time VARCHAR(20)
);
INSERT INTO services (name) VALUES
    ('cut'),
    ('color'),
    ('perm'),
    ('style'),
    ('trim');
EOF
'

# Start the program
main_menu
