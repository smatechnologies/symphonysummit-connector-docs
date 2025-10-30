# Release Notes SymphonySummit

## General

This release of SymphonySummit is for opCon System 21.0 or greater. The connector only supports connections to the OpCon-API to retrieve job information and insert or update Incident information. 

## Release 24.2.0

### New Features

**CON-621**    
					Add capability to assign a task to a specific person using Assigned_Engineer_Email.  

## Release 24.1.0

### New Features

**CONNUTIL-655**    
					Add Incident view url capability to allow full definition of incident viewing URL.  
                    To implement the change, add the new address to the template. This change allows the
                    incident address and the view address to be different if required. 
                    Else define the view-address the same as the main address.

                    },
                    "viewAddress": {
                        "name": "view-address",
                        "value": "address.com"
                    },

## Release 24.0.0

This is the initial release of the SymphonySummit COnnector.

### Migration Considerations

### New Features

### Fixes

