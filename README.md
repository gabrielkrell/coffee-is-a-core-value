# Project Design
#### Local account and balance system for the coffee and soda machines at AEPKS


Justin Schmitz, Daniel Keats (Jimmy)

https://otsi.iit.edu/dropoff/ link for grabbing account names from rfid keys
### Main Objectives:

1. Provide an easy and efficient method for regulating account balances using student's Hawkcards, or any other 125 kHZ RFID card if they are not a student. This includes:

  1. Adding money to accounts.
  2. Purchasing items from the machines.
  3. Refunding account balance.

3. Allow for account management - such that if somebody claims they put money in and didn't get it, or some other strange thing happened, this needs to be fixable by an administrator. This should be easy to do, such that it doesn't need to be done by the guy who wrote the thing, so that it is sustainable into the future.

4. Allow for guests to purchase items - guests need a way to put money into the soda machine and purchase both soda and coffee products.

5. Automatic account registration - such that without having to sign up online or in advance, IIT students will be able to scan their ID for the first time and have an account generate, including pulling their hawk email address.

6. Enforcement of payment - ensure that the system cannot easily be 'broken' so as to allow a person who did not pay to receive product. This would involve a mechanical shield being used on the espresso machine.

7. Encryption - The service should be at least a little secure, with encryption on the administrator passwords.

8. A web portal - a simple website for an administrator where they can access and modify accounts, change prices, and obtain full data exports.

9. A receipt system - a button that, when pressed, sends an email to the user containing their full purchase history, their current balance, and a graph showing their purchases over time.

10. Have an automated email system in place to let users know when their balance is starting to run low (5$, <1$).

11. Account system - Allow for four levels of accounts.
  1. "user": Able to access all standard functionality.
  2. "manager": Able to modify account balances, adjust prices, and produce data exports.
  3. "admin": Full access, including updating the program, access to the databases, and doing basically whatever they want.   

## Workflow:

Task 1: New Registration.

1. Person scans ID.
2. System generates a new account for this person.
3. System finds their hawk name, if available.   

  1. If information is not display a message on the screen with the following: "Unable to find your hawk information. To register your account under your name, follow this link: [google form]. Once you fill out the form, your information will update automatically. Please press OK to continue."

4. Display current balance.

Task 2: Purchasing a coffee.

1. Person scans ID, or is on guest account.
2. The main menu will appear. Prompt the user with their current balance, and the available menu options. "Welcome, [username]. Your current balance is []."
3. The user will then press the 'purchase item' button on the coffee machine terminal.
1. The user's account will be checked, to ensure they have the requisite balance.
2. The user's account will then be deducted the given balance.
3. The plastic shroud covering the touchscreen on the coffee machine will then be lifted, allowing the user to select a drink option.
4. After 30 seconds (arbitrary number), the shroud will then move back into place.

Task 3. Purchasing a soda.

1. Person scans ID, or is on guest account.
2. The main menu will appear. Prompt the user with their current balance, and the available menu options. "Welcome, [username]. Your current balance is []."
3. The user will then press the 'purchase item' button.
4. The user's account will be checked, to ensure they have the requisite balance.
5. The user's account will then be deducted the given balance.
6. [Access control method for soda machine here.]

Task 4. Generating a receipt.

1. Person scans ID, or is on guest account.
2. Depending on machine, user will be welcomed with 'Welcome' state.
3. User will then press the 'receipt' button.
4. If on guest account: User will be told they are on guest account, and return to main menu.
6. If on registered account:

  1. Receipt will be generated, and sent to their email.
  2. Display "Sent a receipt to [email]. To change this address, please fill out [google form]".

Task 5. Payment Systems:

1. All users will accrue balance over a two week long period.
2. At the end of the two week period, users will be emailed invoices to their hawk emails.
3. The users will have one week to pay this.
4. If the user does not pay by the end of the week, their account will be deactivated.

Task 6. Refund:

1. User will press the 'refund' button on the machine.
2. Program will undo the previous transaction for the logged in user.
3. Program will send the user an email, and ask them to explain briefly why they want a refund.
4. Request will be approved/denied by manager.


Managerial tasks:
1. Informing the system managers and administrators when change is running low.
2. Informing the system managers and administrators after a certain number of coffee or soda purchases.
3. Allowing manual overriding of balances, in case something trips up along the line.
4. Approve/deny refund requests.



## Implementation Details:

### Hardware:

Raspberry Pi models will be used for the parts of the machine. The responsibilities of each 'half' of the device:

1. Soda Machine (Master):
   * Host the mySQL server and the web portal.
   * Have an RFID reader for a user to scan into their account.
   * Have a 'receipt' button to send the user their receipt.
   * Have a 'logoff' button, for when the user is done with their session.
   * Have a button for the user to purchase a soda, and regulate this with the account system.


2. Coffee Machine (Slave):
  * Connect to the mySQL server hosted by the Soda Machine Pi.
  * Have an RFID reader for a user to scan into their account.
  * Have a button for the user to purchase a coffee.
  * Have a guard mechanism to uncover and recover the touchscreen of the coffee machine, to prevent unauthorized purchases
  * Have a 'logoff' button, for when the user is done with their session.

### Software:

The program will be mostly written in Python 3. This makes it easy to use mySQL, as well as connecting to physical buttons using GPIO.


#### mySQL Table Design:

The tables will be laid out as follows:

'parameters' table:

| item   | price |
| ------ | ----- |
| coffee | 1.00  |
| soda   | 0.50  |

'machine_information' table:

| timestamp | balance | soda_sold | coffee_sold |
| --------- | ------- | --------- | ----------- |
| *format*  | 69.40   | 31        | 52          |

'users' table:

| rfid_key | user_name | email_address | balance | account_type | soda_price | coffee_price | active |
| -------- | --------- | ------------- | ------- | ------------ | ---------- | ------------ | ------ |
| 113815   | jschmitz2 | i@gmail.com   | 35.25   | admin        | 0.50       | 0.50         | 1      |
| 116381   | jschmoe38 | like@ymail.co | 23.25   | manager      | 0.50       | 0.50         | 1      |
| 114189   | kschmitz2 | coffee@me.com | 05.30   | user         | None       | None         | 0      |

'user' table (for each individual user): (rfid_id_11321)
*note: this table will be named after their user ID*

| timestamp | action   | item | amount | cur_bal |
| --------- | -------- | ---- | ------ | ------- |
| *format*  | purchase | soda | 0.50   | 35.25   |

'sales' table:

| timestamp | rfid_key | item | price |
| --- | --- | --- | ---
| 1551930836.823067 | 113815 | coffee | 1.00

### Requirements:

* Python 3.7.2 or later
* The "mysql-connector-python" module

### Program functions:

* Scan a user's ID card.
* Check the user's ID card against the database, and see if it is present.
* If it is present, set the active account to that user.
* If it is not present, generate a new account for that user, then set the active account to that user.
* Purchase item - purchasing an item removes money from their account, making sure the balance is there, and executes whatever machine.
* Send invoice to email on file.
* Refund balance - returns the user's money from their account.
* Generate receipt - generates a receipt for the user, and sends it to the email on file.
* Update information - updates user information from a Google form.
* Manager control - manually update customer information and balances.
* Manager notifications: Inform the managers when change is running low.

### Program Layout:

**RFID-scanning functions**:
* Scan a user's ID card.

**Account-related functions**:

* Check the user's ID card against the database, and see if it is present.
* If it is present, set the active account to that user.
* If it is not present, generate a new account for that user, then set the active account to that user.

**Account-balance functions**:

* Send invoice to email on file.
* Purchase item - purchasing an item removes money from their account, making sure the balance is there, and executes whatever machine.
* Refund balance - returns the user's money from their account, through manager.

**Manager functions**:
* Manager control - manually update customer information and balances.

**Google form functionality**:
* Update information - updates user information from a Google form.

**Receipt functions**
* Generate receipt - generates a receipt for the user, and sends it to the email on file.
