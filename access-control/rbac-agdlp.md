# Role-Based Access Control using AGDLP

Authorization in the lab environment follows the AGDLP model recommended by Microsoft.

Structure:

Accounts → Global Groups → Domain Local Groups → Permissions

Example implementation:

User
dkim

Global Group
GG_OEE_Employees

Domain Local Group
DL_OEE_Share_Modify

Permission
Modify access to the OEE file share.

This design ensures that users never receive permissions directly.

Permissions are assigned to roles through security groups.
