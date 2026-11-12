# Lab 02 Evidence

## Azure setup

### Resources and roles

|Time (UTC)|Action|Target|Result|Change|
|-|-|-|-|-|
|before 11:09|Create resource group|rg-finance|Success||
|before 11:09|Create resource group|rg-dev|Success||
|11:09:01|Create storage account|stfin680321029 in rg-finance|Success|Standard\_LRS, allowBlobPublicAccess false|
|11:10:20|Create role assignment|Admin on rg-finance|Success|Storage Blob Data Contributor (temporary)|
|11:10:36|Upload blob|q3.txt to container reports|Success||
|11:11:23|Create role assignment|GRP-Finance-Analyst on rg-finance|Success|Storage Blob Data Reader|
|11:11:26|Create role assignment|GRP-Developers on rg-dev|Success|Contributor|
|after 11:11|Delete role assignment|Admin on rg-finance|Success|Temporary data role removed (no output, confirmed absent in the IAM page)|

The IAM pages for rg-finance and rg-dev each showed two assignments: my admin's inherited Owner and the group role above.

## Persona test evidence

### Finance (Fiona)

|Time (UTC)|Action|Result|Detail|
|-|-|-|-|
|\~11:30|List blobs in reports|**Failure**|"You do not have the required permissions needed to perform this operation." (\~19 min after assignment)|
|\~11:34|List blobs in reports (after new login)|Success|q3.txt, BlockBlob, Hot, 16 bytes|
|\~11:35|Upload x.txt to reports|**Denied**|Same "required permissions" error. The text lists every role that allows the action, so it does not mean Reader is invalid, only that no write-capable data role is assigned|

### Developer (Dana)

|Time (UTC)|Action|Result|Detail|
|-|-|-|-|
|\~11:39|Create storage account in rg-dev|**Failure**|"Subscription ... not recognized" and "not found". `az account list` showed only the tenant-level account|
|\~11:43|Open Azure portal as Dana|No subscription shown|"Welcome to Azure! Don't have a subscription?"|
|after 11:43|Open rg-dev in the portal|**Failure**|Error 401, no authorization for `Microsoft.Resources/subscriptions/resourceGroups/read`|
|11:52:13|Create storage account stdev1636879417 in rg-dev (after new login)|Success|provisioningState Succeeded|
|\~11:53|Create storage account in rg-finance|**Denied**|AuthorizationFailed, no authorization for `Microsoft.Storage/storageAccounts/write` over rg-finance|
|\~11:53|Create Reader role assignment on rg-dev|**Denied**|AuthorizationFailed, no authorization for `Microsoft.Authorization/roleAssignments/write` over rg-dev|

### Help desk (Hannah)

|Time (UTC)|Action|Result|Detail|
|-|-|-|-|
|\~11:57|Reset password for Pat|Success|"Password has been reset", temporary password hidden in the screenshot|
|\~12:00|Reset password for admin account|Not counted|Signed in as the admin, not Hannah, so this does not test her role|
|\~12:02|Reset password for admin account as Hannah|**Denied**|"The password can not be reset. This may be due to an incorrect level of administrative privilege or if trying to reset your own password." Edit properties and Delete were greyed out. The top right showed Hannah's account|

## Sign-in evidence

### Azure CLI sign-in errors

|Time (UTC)|Account|Method|Result|Error code|Meaning|
|-|-|-|-|-|-|
|10:36|Admin|az login in Cloud Shell|Blocked|530035|Not confirmed. Sign-in logs were too slow to load|
|10:51|Admin|az login (device code)|Blocked|530035|Not confirmed|
|11:25|Fiona (account not shown on the error page)|az login --use-device-code|Blocked|530035|Not confirmed. A normal browser login worked afterwards|

App name on every error was Microsoft Azure CLI. Browser logins with `az login --allow-no-subscriptions` succeeded for Fiona and Dana.

## Validation query

Run in Cloud Shell as admin at \~12:04 UTC:

&#x20;   az role assignment list --all --query "\[?principalType=='User'].{who:principalName,role:roleDefinitionName,scope:scope}" -o table


|Who|Role|Scope|
|-|-|-|
|admin account|Owner|/subscriptions/<subscription-id>|

No user holds a role on rg-finance or rg-dev. The admin's Owner is inherited from the subscription.

