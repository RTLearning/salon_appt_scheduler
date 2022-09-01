# salon_appt_scheduler
Bash &amp; Postgres Database- Salon Appointment Scheduler
SALON Appointment Scheduler using BASH Scripting and Postgres

In the database Salon, there are three tables which are used for this app.

**Customers**
customer_id (Serial, PK)
phone(varchar, Unique, Not Null)
name(varchar, Not Null)


**Services**
service_id (Serial, PK)
name (varchar, Unique, Not Null)


**Appointments**
appointment_id (Serial, PK)
customer_id(int, not null, FK)
service_id(int, not null, FK)
time (varchar)



*** BASH SCRIPT ****

#!/bin/bash

PSQL="psql -X --username=freecodecamp --dbname=salon --tuples-only -c"

echo -e "\n~~~~~ Welcome to My Salon ~~~~~"

MAIN_MENU()
{
  if [[ $1 ]]
  then
    echo -e "\n$1"
  fi
  
  echo -e "\nHow can I help you?"
  SERVICE_MENU
  MAX_SERVICE_ID=$($PSQL "select max(service_id) from services")
  echo -e "$((MAX_SERVICE_ID+1))) EXIT"



 
  echo -e "\nPlease enter a service id from above"
  read SERVICE_ID_SELECTED
  if [[ ! $SERVICE_ID_SELECTED =~ ^[0-9]+$ ]]
  then
     MAIN_MENU "That is not a valid service id."
  elif [[ $SERVICE_ID_SELECTED -ge 1 && $SERVICE_ID_SELECTED -le $MAX_SERVICE_ID ]]
  then
      SERVICE_PROCESS $SERVICE_ID_SELECTED
  elif [[  $SERVICE_ID_SELECTED == $((MAX_SERVICE_ID+1)) ]] #exit program
  then
      EXIT
  fi
  
  
}

SERVICE_MENU()
{
  #get available services
  AVAILABLE_SERVICES=$($PSQL "select service_id, name from services order by service_id")
  #echo "$AVAILABLE_SERVICES" by placing the variable within quotes, will display rows
  echo "$AVAILABLE_SERVICES" | while read SERVICE_ID BAR SERVICE_NAME
  do
    echo "$SERVICE_ID) $SERVICE_NAME"
  done

}
SERVICE_PROCESS()
{
      SERVICE_ID=$1

      CUSTOMER_EXISTS=0

      echo -e "\nWhat's your phone number?"
      read CUSTOMER_PHONE
      CUSTOMER_NAME=$($PSQL "select name from customers where phone='$CUSTOMER_PHONE'")
      CUSTOMER_NAME=`echo $CUSTOMER_NAME | sed 's/ *$//g'` # sed removes the trailing spaces
      
      # if customer record is not available
      if [[ -z $CUSTOMER_NAME ]]
      then
        echo -e "\nI don't have a record for that phone number, what's your name?"
        read CUSTOMER_NAME
        #insert new customer
        INSERT_CUSTOMER=$($PSQL "insert into customers(phone,name) values('$CUSTOMER_PHONE','$CUSTOMER_NAME')")
        if [[ $INSERT_CUSTOMER == "INSERT 0 1" ]]
        then
          CUSTOMER_EXISTS=1
        fi
      else
        CUSTOMER_EXISTS=1
      fi
      #if customer records are found / created
      if [[ $CUSTOMER_EXISTS == 1 ]] 
      then
          CUSTOMER_ID=$($PSQL "select customer_id from customers where phone='$CUSTOMER_PHONE'")
          
          SERVICE_NAME=$($PSQL "select name from services where service_id=$SERVICE_ID")
          SERVICE_NAME=`echo $SERVICE_NAME | sed 's/ *$//g'` # sed removes the trailing spaces
          
          # request for service time and insert details into appointment table with cust_id, service_id & service time
          echo -e "\nWhat time would you like your $SERVICE_NAME, $CUSTOMER_NAME?"
          read SERVICE_TIME
          INSERT_APPOINTMENT=$($PSQL "insert into appointments(customer_id,service_id,time) values($CUSTOMER_ID,$SERVICE_ID_SELECTED,'$SERVICE_TIME')")
          if [[ $INSERT_APPOINTMENT == "INSERT 0 1" ]]
          then
            echo -e "\nI have put you down for a $SERVICE_NAME at $SERVICE_TIME, $CUSTOMER_NAME."
          else
            MAIN_MENU "Could not make an appointment, please try again"

          fi
      else
          MAIN_MENU "Customer details doesn't exist / created, please try again" 
      fi
}

EXIT()
{
   echo -e "Thank you for contacting us.\n"
}
MAIN_MENU
