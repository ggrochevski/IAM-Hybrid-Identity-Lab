# Kerberos Authentication Validation

The Active Directory environment uses Kerberos as its authentication protocol.

When a user logs into the domain-joined workstation, the Domain Controller validates the user's credentials and issues a Ticket Granting Ticket (TGT).

The TGT allows the client to request service tickets without re-entering credentials.

Kerberos tickets were validated using the command:

klist

Example output showed the following tickets:

krbtgt/TREASURY.LOCAL
cifs/DC01

The TGT confirms successful authentication to the domain.

The CIFS service ticket appeared after accessing the network file share hosted on the Domain Controller.

This confirms that Kerberos authentication is functioning correctly in the lab environment.
