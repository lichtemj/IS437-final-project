# IS437 Final Project: Web App Design
This is going to be a mockup Employee Management Suite for a generic company (note that I did not look at the sample topics first, but after seeing the time clock I thought it to be a bit simple). There will be 3 components: A SQL database, a Python application, and an HTTP interface. 
## SQL Databse Breakdown
The SQL database will have the following tables:
- Employee (id, name, date of birth, wage offset, job id as fk)
- Job (id, title, wage, administrator)
- Schedule (id, date, in, out, employee as fk)
- TimeClock (id, date, in, break, return, out, shift length, employee id as fk)
- Payroll

Each table has different attributes relative to its function (See reltional schema). The TimeClock and Payroll tables have calculated fields for ease of use, and Schedule is used to differentiate between break and bookend punches.
## Menu & Logic Breakdown
Here is a list of each menu I plan to have in this app, along with the logic and result of each menu. Note: for functions requiring configuration setup, this will be stored in a file at the application level, as it will be invoked to block actions at the application and database levels. A flag for whether or not the user has administrator permissions in this screen is stored in the database.
### Screen: Punch Clock
- Given By System: Current date, time
- User Input: Employee ID
- User Action: Punch
### Logic and Results
- Condition: Punch outside of valid interval range
  - Check for existing record from employee on today's date
      - if exists: write punch as break or return based on null values
        - if break is null, write time to break
        - if break is not null and return is null, write time to return
        - if both are not null, block the punch and alert the user that they cannot punch at this time
      - if not exists: block punch
- Condition: Punch within valid interval of scheduled time in
  - Check for employee record from today's date
    - if not exists: create record for employee on this date, use time as in punch
    - if exists: block punch, tell user they have already punched in today
- Condition: Punch within valid interval of scheduled time out
  - Check for employee record from today's date
    -if exists: check field out_time
      -if null: write timestamp, calculate shift length
      -if not null: block punch, tell user they have already punched out today
    -if not exists: block punch, tell user they cannot punch out if they haven't punched in
### Screen: Administrator Login (Intermediate screen thrown by all managerial tasks)
- Given By System: List of Users With Administrator Permissions (via job title)
- Required By System: Screen to Redirect to, Screen redirected from
- User Input: Employee ID
- User Action: Login, Return
### Logic And Results
- Condition: User attempts login
  - Check whether or not user has administrator permissions (Only users with admin perms are required to log in to access functionality)
    - if yes: redirect to target screen
    - if no: block login attempt, tell user they cannot access this functionality
- Condition: User returns to previous screen
  - Redirect user to stored previous screen
### Screen: Schedule Manager (Login Required)
- Given by System: Existing Schedule
- User Input: Employee ID, Date, Time In (Submit Only), Time Out (Submit Only)
- User Action: Submit, View
### Logic And Results
- Condition: User views employee schedule for a certain day
  - if employee is scheduled: view schedule for employee for specified date
  - if employee is not scheduled: tell user employee was not scheduled
- Condition: User provides date, but no employee ID
  - returns all employees scheduled for that day, in ascending order of start time (earliest first)
- Condition: User submits new schedule info
  - Check for existing record matching employee and date parameters
    - if exists: update in and out time if both are provided, otherwise only update the fields where the form is not null
    - if not exists: check for both in and out time
      - if neither are null: create record
      - if one or more fields are null: block insertion, alert user
### Screen: Employee Manager (Login Required)
- Given by System: List of Employees
- User Input: Employee ID, Name, Date of Birth, Wage Offset
- User Action: Submit
### Logic And Results
- User submits info
  - Check if employee exists (check by id)
    - if exists: update fields if form field is not null (if null then ignore)
    - if not exists: Check fields
      - if all not null: create record, ignore ID field (Auto-Increment)
      - if one or more null: block insert, alert user that no fields may be null for new employee
### Screen: Job Manager (Login Required)
- Given by System: Job list
- User Input: Job ID, Title, Hourly Wage, Administrator Flag
- User Action: Submit, View
### Logic And Results
- Condition: User submits info
  - Check if job exists using ID
    - if exists: update using not null form fields
    - if not exists: Check for form completeness
      - if all fields filled out: create record using fields (ignore ID, have database assign it)
      - if one or more fields null: block insertion, alert user
### Screen: Punch Manager (Login Required)
- Given by System: Record of Punches
- User Input: Employee ID, Punch Type (in, break, return, out), Time
- User Action: Submit
### Logic And Results
- Condition: User Submits Info
  - Check schedule for valid placement
    - if valid: sumbit punch
    - if invalid: block punch, tell user to enter valid time or modify schedule
  - Check if both in and out are not null
    - if true: calculate shift length
    - if false: leave shift length uncalculated
### Screen: Payroll Viewer (Login Required)
- Given by System: Pay info by employees
- User Input: Pay Period Start
- User Action: Submit
### Logic And Results
- Condition: User Sumbits Info
  - Check if employee pay for period is nonzero
    - if true: include in return list
    - if false: do not include in return dataset
### Screen: Configuration (Login Required)
- Given by System: Current Config Structure
- User Input: Acceptable Punch Interval (Min), Pay Period Length (weeks), Pay Period Start Day, Pay Day
- User Action: Submit, View
### Results
- Condition: User Submits
  - Write info to config structure
- Condition: User Views Info
  - Print current config structure
## Additional Notes
This is merely an outline of the project plans, this will be updated as the project comes together.
