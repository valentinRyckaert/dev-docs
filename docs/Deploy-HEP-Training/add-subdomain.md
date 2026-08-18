# Add a subdomain on OKD for a space

On OKD, with de "Core Platform" view, select "Networking" > "Routes".

Click on "Create Route" and fill as following:

- Name: whatever you want
- Hostname: the subdomain you want to create (can be anything but must finish with ".app.cern.ch")
- Path: leave it blank
- Service: select the main application service
- Target Port: select the one which your HEP Training is running on
- enable Secure Route
- TLS termination: Edge
- Insecure traffic: Redirect
- leave the rest blank as it is

Save your route. You don't need to restart your deployment.