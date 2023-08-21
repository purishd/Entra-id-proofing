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

# Build a Powerautomate flow
This is how my Powerautomate flow looke like.
![image](https://github.com/purishd/Helpdesk/assets/11908199/caa34969-084f-4273-8dea-8a3345774e4f)

# Audit record for OTP
