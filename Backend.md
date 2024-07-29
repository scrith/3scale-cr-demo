# Backend Custom Resource

The resources will create will be done in a specific order as there are dependencies on each other. The first we will create is the Backend CR. The YAML for a basic backend will look like this:

~~~
apiVersion: capabilities.3scale.net/v1beta1
kind: Backend
metadata:
  name: weather-alerts-backend
  namespace: 3scale
spec:
  name: Weather Alerts API Backend
  systemName: weather-alerts-backend
  privateBaseURL: https://api.weather.gov:443
  description: Backend of Weather Alerts API
~~~

* The fields inside `metadata` provides the OCP object details.
* The fields inside `spec` provides the 3scale specific details

~~~
$ oc apply -f weather-backend.yml 
backend.capabilities.3scale.net/weather-alerts-backend configured
~~~

The creation process is very fast and OCP will return you to the the listing of Backend CRs. If the "Status" field displays "Condition: Synced" then the creation of the backend is successful and we can verify in the 3scale Admin console.

The complete spec for all the options in the Backend CR can handle can be found in the operator documentation here: https://github.com/3scale/3scale-operator/blob/3scale-2.14-stable/doc/backend-reference.md

If changes are added to the design of the backend, it is very easy to update the Backend defintion in 3scale by updating the CR yaml file and using `oc apply -f` again. The system will recognize it is the same object as previously deployed and update the information associated with it.

Update the yaml file to include an additional method that can be used for tracking by appending the following:

~~~
  methods:
    sameplemethod:
      friendlyName: New Sample Method
      description: Additional Method to use for tracking.
~~~

Apply the new configuration:

~~~
$ oc apply -f weather-backend-updated.yml 
backend.capabilities.3scale.net/weather-alerts-backend configured
~~~

You are now ready to [create a product](Product.md) that utilize this backend.