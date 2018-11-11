Regenerate https certificate for application

0) Clean all previous https certificates

dotnet dev-certs https --clean

1) Go to project directory

cd CM.Services.Identity.API

2) Check if project currently contains password for https cerificate

dotnet user-secrets list

Run command and check for key "Kestrel:Certificates:Development:Password".
If password exists then copy it and go for 4, if not go for 3.

3) Create password for https certificate

dotnet user-secrets set "Kestrel:Certificates:Development:Password" "07985ce2-f74b-402e-b7d9-16bf78fe4bcc"

4) Create certificate

dotnet dev-certs https -ep C:\Users\verim1990\AppData\Roaming\ASP.NET\Https\CM.Services.Identity.API.pfx -p 07985ce2-f74b-402e-b7d9-16bf78fe4bcc

5) Repeat steps from 1 to 4 for all microservices, remembering abour changing guids and project names

6) Make all created certificates trusted

dotnet dev-certs https --trust