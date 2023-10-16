# Entra Identity Proofing

This repo is created to help build automation that makes helpdesk job easier.

| Title | Description |Author|
|-----:|---------------|-----|
| Identity Proofing Made Easy|This doc will help customers set up Identity Proofing application which is often required by their helpdesk teams while verifying people when they call helpdesk with any request.               |purishd|

# Glossary of Acronyms

| Acronym | Description |Comment|
|-----:|---------------|-----|
| MVP|Minimum Viable Product              |It is simlary to widely known term OTP|
| IPC|Identity Proofing Code               |It is simlary to widely known term OTP|

# Disclaimer
Please note that we are aware of security risks pertaining to use of single use one-time codes. However, the one-time code used in this solution is not tied to any authentication service as such, hence unusable for any sort of attack. A term IPC is being used which is similar to well known term One time password(OTP).

The purpose of building this is to show a way to leverage the low code approach to build a quicker Identity Proofing solution. This solution is currently a MVP that one can leverage, extend it and build something of their own. The documentation will evolve as we build more scenarios into it as we have more ideas waiting to be built into this solution.
Few scenarios that we are looking to build further into this are:

1. Share TAP(Temporary Access Pass) with end users post Identity Proofing.
2. Use the reverse flow to do Identity Proofing when helpdesk initiates a call to the end user.
3. A few more as we learn.

# Problem Statement
Often customer's end users call their helpdesk team to fix any issue for them such us updating their user profile, password reset etc. The helpdesk team needs to verify the users. For that purpose, they ask the end users some of their personal information such as DOB, drivers license, employee ID etc. to confirm the user's Identity. This is an Identity Proofing scenario that need not rely on personal information necessarily but something super simple. If something as simple as sending an IPC to the calling user on their mobile and asking them to verify the same back with the helpdesk could be a potential solution, then the help desk don't need to necessarily delve into user's personal information.

# Solution

Build a super simple application that will generate IPC and send it to the user. Helpdesk can verify the user by requesting them for the same OTP.

Watch the video below to understand the high level flow.

https://github.com/purishd/Helpdesk/assets/11908199/118b8c93-3f73-41f8-9e82-2c85d3f2c601


# Detailed Design

Here is a sequence diagram representing the detailed flow.

![image](/assets/SequenceDiagram.png)

From technical implementation perspective, the sequence diagram above translates into following high level steps.
1. Build a Power App UI that will collect user's UPN. This is the front-end that will be used by helpdesk person to send IPC on user's mobile.
2. Build Power Automate flow that will read the user's mobile number and send IPC using Azure Communication Services.
3. Record this IPC in a SharePoint List for verification and auditing purposes.

## Objective

Build a Power App that can be used for Identity proofing by your helpdesk team.

## Prerequisites
These are the resources that are needed to be set up before starting to build the Power App and Power Automate flow.
## Set up Azure Communication services
An "Communication services" resource has been set up in Azure subscription, an Alphanumeric sender ID is enabled for one-way outbound SMS used for sending Identity Proofing Code (IPC) to user's mobile number. 

*Create communication services resource:* https://learn.microsoft.com/en-us/azure/communication-services/quickstarts/create-communication-resource?tabs=windows&pivots=platform-azp

*Enable Alphanumeric Sender ID:* https://learn.microsoft.com/en-us/azure/communication-services/quickstarts/sms/enable-alphanumeric-sender-id

## Create a SharePoint connection in Power Automate Data connections
This connection is used to update list item on SharePoint list for audit purpose.
![image](/assets/SharePointConnector.png)
# Build a Power App

This is how my sample Power App looks like. Follow below guidelines to create a similar Power App or use your own creativity to extend this Power App and make it even better.

![image](/assets/PowerappUI.png)

Following controls are being used in this super simple app.
1. An Image for customer logo.
2. Label for the name of the app.
3. Label and text input to get the user UPN.
4. Button to send the IPC that will hook into my Power Automate flow.

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
   Setting the "Text" property of this button to variable newOTP. This variable is blank initially but its value is set when the Power Automate flow sends the OTP back to the Power App.
  
6. A reset button that I am using to quickly reset the values of controls. Feel free to use any other ways as you like to reset the form values.
```
   Reset(UPNTextInput);
   Set(newOTP,"")
```


# Build a Power Automate flow
Follow below steps to create Power Automate Flow. This is how your overall flow will look like once all the steps are added to it.

![image](/assets/FlowOverall.png)


1. Create a new instant cloud flow.
2. Choose PowerApps as trigger of this flow.
   
   ![image](/assets/PowerappStart.png)

4. Intialize a variable that will collect the user UPN value from Power Apps.
   ![image](/assets/InitializeVariable.png)

5. Get a graph token using the client credential flow. You can choose other ways to get a graph token as you prefer.
   
   ![image](/assets/GraphToken.png)

7. Parse the JSON output from above step to get the access token.

   ![image](/assets/ParseGraphToken.png)

9. Make a Graph API call to get user's mobile number from user's authentication method.

    ![image](/assets/GetMobileAuthMethod.png)

11. Parse the JSON output from above step to get the mobile number.

    ![image](/assets/ParseGetMobileAuthMethod.png)

13. Add compose action to get value of mobile number.

    ![image](/assets/ComposeMobile.png)

15. Generate IPC using random generator function. You can choose your own OTP generator as you prefer.

    ![image](/assets/GenerateOTP.png)

17. If mobile number from step 8 above is not null, then use Azure communication Resources connector to send an Identity Proofing Code (IPC) on the mobile number.

    ![image](/assets/ConditionMobileNotNull.png)

    Select Azure communication service SMS action.

    ![image](/assets/SendSMS.png)

    To do above, you could follow below steps to create a new ACS SMS connection. Once the connection is created, you can follow above steps to configure ACS to send IPC on user's mobile number.
    
    - For this sample, keys have been used for authentication. Go to Azure communication services resource keys section and copy the connection string.      
      ![image](/assets/ACSSendSMSAzurePortal.png)
      
    - Create a new connection in Power Automate. Select Azure communication service SMS (Premium) and select authentication type (connection string or AAD application authentication) as you prefer. Use the connection string from previous step to create a successful connection.
      
      ![image](/assets/ACSConnectorFlow.png)


19. Send the generated OTP to Power App in a variable.

    ![image](/assets/RespondtoPowerApp.png)


# Audit record for IPC
For audit record, a SharePoint list is being used. In Power Automate flow use Create item action of SharePoint to add an audit entry on your chose SharePoint list.

![image](/assets/SharePointCreateItem.png)

This is how the SharePoint record will look like. You could choose to add/remove/modify fields as per your preference.

![image](/assets/SharePointList.png)

