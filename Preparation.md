# Preparation Work

## Tenant Identification

In a multi-tenant environment you will need to create a secret to be used by the CRs to know which tenant to deploy the resources to. The tenant CR can be created with a yaml file directly with `oc`. The yaml will look like the one below:

~~~
apiVersion: v1
kind: Secret
metadata:
  name: 3scale-tenant-secret
type: Opaque
stringData:
  adminURL: https://3scale-admin.apps.phagerma.lab.upshift.rdu2.redhat.com:443
  token: "16746d835bb600c5923bada229089b74f3a81805af1af9e4f5befc02605333cb"
~~~

**Note:** Update the `adminURL` and `token` with values from your system. The token will need to have Read/Write access to the *Account Management API* scope.

Install this secret to your system in the same namespace as 3scale is deployed.

~~~
$ oc apply -f tenant-secret.yml 
secret/3scale-tenant-secret created
~~~

Now that the preparation is complete, continue to creating a [backend](Backend.md)