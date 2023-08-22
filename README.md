# Helpdesk
This repo is created to help build automation that makes helpdesk job easier.

| Title | Description |Author|
|-----:|---------------|-----|
| Helpdesk Identity Proofing|This doc wil help customers set up Identity Proofing application which is required by their helpdesk teams while identifying people when they call helpdesk with any request               |purishd|

# Problem Statement
Often customer's end users call their helpdesk team to fix any issue for them such us updating their user profile, password reset etc. The helpdesk team needs to verify the users. For that purpose, they ask the end users some of their personal information such as DOB, drivers license, employee ID etc. to confirm the user's Identity. This is an Identity Proofing scenario that need not rely on pesonal information necessarily but something super simple. If somehting as simple as sending an OTP to the calling user on their mobile and ask them to verify the same back with the helpdesk could be a potential solution, then the help desk don't need to necessarily delve into user's personal information.

# Solution
Build a super simple application that will generate OTP and send it to the user. Helpdesk can verify the user by requesting them for the same OTP.

# Objective
Build a Powerapp that can be used for Identity proofing.

# Detailed Design
1. Build a Powerapp UI that will collect user's UPN. This is the front-end that will be used by heldesk person to send OTP on user's mobile.
2. Build Powerautomate flow to that will read the user's mobile number and send OTP using Azure Communication Services.
3. Record this OTP in a SharePoint List for verification and auditing purposes.

# Build a Power App

This is how my sample Power App looks like.

![image](https://github.com/purishd/Helpdesk/assets/11908199/e9891599-a7a3-43fc-877e-3efcd864e1d0)

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
This is how my Powerautomate flow looke like.
![image](https://github.com/purishd/Helpdesk/assets/11908199/caa34969-084f-4273-8dea-8a3345774e4f)

1. Create a new instant cloud flow.
2. Choose PowerApps as trigger of this flow.
3. Intialize a variable that will collect the user UPN value from Power Apps.
4. Get a graph token using the client credential flow. You can choose other ways to get a grpah token as you prefer.
5. Parse the JSON output from above step to get the access token.
6. Make a Grpah API call to get user's mobile number from user's authentication method.
7. Parse the JSON output from above step to get the mobile number.
8. Add compose action to get value of mobile number.
9. Generate OTP using random generator function. You can choose your own OTP generator as you prefer.
10. If mobile number from step 8 above is not null, then use Azure communication Resources connector to send an OTP on the mobile number. Select Azure communication service SMS action.

    - Create a new connection in Power Automate. Select Azure communication service SMS (Premium) and select authentication type (connection string or AAD application authentication)
    ![image](https://github.com/purishd/Helpdesk/assets/11908199/03a68788-a352-4d4d-b258-33b0425d8c76)

12. Send the generated OTP to Power App in a variable.


# Audit record for OTP
