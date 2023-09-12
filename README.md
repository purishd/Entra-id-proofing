# Helpdesk
This repo is created to help build automation that makes helpdesk job easier.

| Title | Description |Author|
|-----:|---------------|-----|
| Helpdesk Identity Proofing|This doc wil help customers set up Identity Proofing application which is required by their helpdesk teams while identifying people when they call helpdesk with any request               |purishd|

# Disclaimer
The purpose of this is to show a way to leverage the low code approach to build something quicker. This solution is currently a Minimum Viable Product (MVP) that one can leverage, extend it and build something of their own. This doucmentation will evolve as we buid more scenarios into it as we have more ideas waiting to be built into this solution.

# Problem Statement
Often customer's end users call their helpdesk team to fix any issue for them such us updating their user profile, password reset etc. The helpdesk team needs to verify the users. For that purpose, they ask the end users some of their personal information such as DOB, drivers license, employee ID etc. to confirm the user's Identity. This is an Identity Proofing scenario that need not rely on pesonal information necessarily but something super simple. If something as simple as sending an OTP to the calling user on their mobile and asking them to verify the same back with the helpdesk could be a potential solution, then the help desk don't need to necessarily delve into user's personal information.

# Solution
Build a super simple application that will generate OTP and send it to the user. Helpdesk can verify the user by requesting them for the same OTP.

![image](https://github.com/purishd/Helpdesk/assets/11908199/fb3af27e-bf35-4a36-9e92-aa1a66cae03a)


# Objective
Build a Powerapp that can be used for Identity proofing by customer helpdesk team.

# Detailed Design
1. Build a Powerapp UI that will collect user's UPN. This is the front-end that will be used by heldesk person to send OTP on user's mobile.
2. Build Powerautomate flow that will read the user's mobile number and send OTP using Azure Communication Services.
3. Record this OTP in a SharePoint List for verification and auditing purposes.

# Prerequisites
These are the resources that are needed to be set up before starting to build the Power App and Power Automate flow.
## Set up Azure Communication services
An "Communication services" resource has been set up in Azure subscription, an Alphanumeric sender ID is enabled for one-way outbound SMS used for sending OTP to user's mobile number. 

*Create communication services resource:* https://learn.microsoft.com/en-us/azure/communication-services/quickstarts/create-communication-resource?tabs=windows&pivots=platform-azp

*Enable Alphanumeric Sender ID:* https://learn.microsoft.com/en-us/azure/communication-services/quickstarts/sms/enable-alphanumeric-sender-id

## Create a SharePoint connection in Power Automate Data connections
This connection is used to update list item on SharePoint list for audit prupose.
![image](https://github.com/purishd/Helpdesk/assets/11908199/f996acdc-3799-428c-a980-193b935ef053)
# Build a Power App

This is how my sample Power App looks like.

![image](https://github.com/purishd/Helpdesk/assets/11908199/2802f697-71cc-4a90-9b78-2acbc1ea391a)




I have following controls in this super simple app.
1. An Image for customer logo.
2. Label for the name of the app.
3. Label and text input to get the user UPN.
4. Button to send the OTP that will hook into my Power Automate flow.

Calling below code on "OnSelect" property of the Send OTP button. This code will help set the buttonPressed and then call Power Automate flow and set the value of variable newOTP to generated OTP.
```
Set(buttonPressed,"sendOTPButton");
Set(newOTP,SMSFlow.Run(UPNTextInput.Text).otp);
```

5. A hidden label that only lights up when the OTP is generated and shows the generated OTP on the screen for helpdesk person to view it quickly.

   Making this label visible when send OTP button is pressed.
```
   If(buttonPressed = "sendOTPButton", true, false)
```
   Setting the "Text" property of this button to variable newOTP. This variable is blank initially but its value is set when the Power Autoate flow sends the OTP back to the Power App.
  
6. A reset button that I am using to quickly reset the values of controls. Feel free to use any other ways as you like to reset the form values.
```
   Reset(UPNTextInput);
   Set(newOTP,"")
```


# Build a Powerautomate flow
This is how the Powerautomate flow looks like.
![image](https://github.com/purishd/Helpdesk/assets/11908199/2f4ed1ca-d671-4401-b875-ecc83ed7d062)


1. Create a new instant cloud flow.
2. Choose PowerApps as trigger of this flow.
   
   ![image](https://github.com/purishd/Helpdesk/assets/11908199/182114f9-571d-41ed-869f-ea5900b51320)

4. Intialize a variable that will collect the user UPN value from Power Apps.
   ![image](https://github.com/purishd/Helpdesk/assets/11908199/edfb4b98-682f-456e-9d05-c95336dff09c)

5. Get a graph token using the client credential flow. You can choose other ways to get a graph token as you prefer.
   
   ![image](https://github.com/purishd/Helpdesk/assets/11908199/0228d6d2-40f6-4d35-9901-f9a733f7c602)

7. Parse the JSON output from above step to get the access token.

   ![image](https://github.com/purishd/Helpdesk/assets/11908199/eece5f1f-44ec-4cc6-a3d7-342aa4f6773c)

9. Make a Graph API call to get user's mobile number from user's authentication method.

    ![image](https://github.com/purishd/Helpdesk/assets/11908199/f617440d-6479-489b-a502-0914b58e2c8c)

11. Parse the JSON output from above step to get the mobile number.

    ![image](https://github.com/purishd/Helpdesk/assets/11908199/9546c676-328e-4a63-adc5-a5dff4580bb8)

13. Add compose action to get value of mobile number.

    ![image](https://github.com/purishd/Helpdesk/assets/11908199/aea4c72e-895a-4503-a35d-80f76c9ec7e1)

15. Generate OTP using random generator function. You can choose your own OTP generator as you prefer.

    ![image](https://github.com/purishd/Helpdesk/assets/11908199/cbb6520c-be58-4768-b55b-e610e9953b14)

17. If mobile number from step 8 above is not null, then use Azure communication Resources connector to send an OTP on the mobile number.

    ![image](https://github.com/purishd/Helpdesk/assets/11908199/3e9d1786-f202-41e5-ab91-99562c5edd3e)

    Select Azure communication service SMS action.

    ![image](https://github.com/purishd/Helpdesk/assets/11908199/be16cd13-ab7b-403b-a991-b048f74a0c4c)

    To do above, you could follow below steps to create a new ACS SMS connection. Once the connection is created, you can follow above steps to configure ACS to send OTP on user's mobile number.
    
    - For this sample, keys have been used for authentication. Go to Azure communication services resource keys section and copy the connection string.      
      ![image](https://github.com/purishd/Helpdesk/assets/11908199/da5c0870-327d-4ee4-9ccb-f4583073728e)
      
    - Create a new connection in Power Automate. Select Azure communication service SMS (Premium) and select authentication type (connection string or AAD application authentication) as you prefer. Use the connection string from previous step to create a successful connection.
      
      ![image](https://github.com/purishd/Helpdesk/assets/11908199/03a68788-a352-4d4d-b258-33b0425d8c76)


19. Send the generated OTP to Power App in a variable.

    ![image](https://github.com/purishd/Helpdesk/assets/11908199/ddc9f752-7533-49aa-bd82-12df9fd43e38)


# Audit record for OTP
For audit record, a SharePoint list is being used. In Power Automate flow use Creat item action of SharePoint to add an audit entry on your chose SharePoint list.

![image](https://github.com/purishd/Helpdesk/assets/11908199/828917db-035c-405b-bae9-e42b5389a98a)

This is how the SharePoint record will look like. You could choose to add/remove/modify fields as per your preferce.

![image](https://github.com/purishd/Helpdesk/assets/11908199/f62a8c4a-be07-4471-93ca-72a1fe37bc09)

