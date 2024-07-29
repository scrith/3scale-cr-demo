# Product

The basic outline for a product that uses the backend we've just created is as follows:

~~~
apiVersion: capabilities.3scale.net/v1beta1
kind: Product
metadata:
  name: weather-alerts-api
  namespace: 3scale
spec:
  name: Weather Alerts API Product
  systemName: weather-alerts-api
  providerAccountRef:
    name: 3scale-tenant-secret
  description: Get state-level weather alerts
  mappingRules:
  - httpMethod: GET
    pattern: "/"
    metricMethodRef: hits
    increment: 1
    last: false
  metrics:
    hits:
      friendlyName: Hits
      unit: hit
      description: Number of API hits
  methods: {}
  policies:
  - name: apicast
    version: builtin
    configuration: {}
    enabled: true
  applicationPlans: {}
  backendUsages:
    weather-alerts-backend:
      path: "/"
~~~

This will create a very basic API Product, but it will not be fully functional yet. We will need to add some additional details to the yaml and update the CR. We can start by defining a new entry in our policy chain that we know will be required for this API to make successful calls to the backend.

Update the `policies` section with the additional policy to remove the `user_key` after processing by the gateway, but before passing the request to the backend. For that we will utilize the `url_rewriting` policy.

~~~
  policies:
  - name: apicast
    version: builtin
    configuration: {}
    enabled: true
  - name: url_rewriting
    version: builtin
    configuration:
      query_args_commands:
      - value_type: plain
        op: delete
        arg: user_key
    enabled: true
~~~

We can also define the methods we plan to use for tracking the calls to the API by defining the `methods` yaml section:

~~~
  methods:
    activealertsbyarugment:
      friendlyName: ActiveAlertsByArugment
      description: Returns a list of alerts based on state as an argument.
    activealertsbyurl:
      friendlyName: ActiveAlertsByURL
      description: Returns a list of alerts based on state as a url.
    echo:
      friendlyName: Echo
      description: Echoes back request details
~~~

Apply the changes to update the CR.

~~~
$ oc apply -f weather-product-update-1.yml 
product.capabilities.3scale.net/weather-alerts-api configured
~~~

Now that we've configured the methods for tracking, we can next create the Mapping Rules, by updating the `mappingRules` section:

~~~
  mappingRules:
  - httpMethod: GET
    pattern: "/alerts/active$"
    metricMethodRef: activealertsbyarugment
    increment: 1
    last: false
  - httpMethod: GET
    pattern: "/alerts/active/area/{state}$"
    metricMethodRef: activealertsbyurl
    increment: 1
    last: false
  - httpMethod: GET
    pattern: "/echo$"
    metricMethodRef: echo
    increment: 1
    last: false
~~~

Apply the changes to update the CR.

~~~
$ oc apply -f weather-product-update-2.yml 
product.capabilities.3scale.net/weather-alerts-api configured
~~~

The last required part of the definition to update is to make an Application Plan for the Product. Update the `applicationPlans` section with this definition:

~~~
  applicationPlans:
    weather-alerts-plan:
      name: Weather Alerts Plan
      appsRequireApproval: false
      trialPeriod: 0
      setupFee: "0.00"
      custom: false
      state: published
      costMonth: "0.00"
      pricingRules: []
      limits: []
~~~

Apply the changes to the updated CR.

~~~
$ oc apply -f weather-product-update-3.yml 
product.capabilities.3scale.net/weather-alerts-api configured
~~~

The final change we will make in this demo is to add an additional policy to route specific requests to a different service. Update the `policies` section as follows, adding a `routing` policy above `apicast`:

~~~
  policies:
  - name: routing
    version: builtin
    configuration:
      rules:
      - condition:
          combine_op: and
          operations:
          - value_type: plain
            value: "/echo"
            match: path
            op: matches
        url: https://echo-api.3scale.net:443
    enabled: true
  - name: apicast
    version: builtin
    configuration: {}
    enabled: true
  - name: url_rewriting
    version: builtin
    configuration:
      query_args_commands:
      - value_type: plain
        op: delete
        arg: user_key
    enabled: true
~~~

This new policy will route requests that match "/echo" to the 3scale.net echo-api service.

An optional step that is useful in many environments is to update the URLs for the staging and production endpoints. This can easily be accomplished by appending this code to the end of the yaml for thr Product CR:

~~~
  deployment:
    apicastSelfManaged:
      stagingPublicBaseURL: "https://weather-api-staging.apps.phagerma.lab.upshift.rdu2.redhat.com"
      productionPublicBaseURL: "https://weather-api.apps.phagerma.lab.upshift.rdu2.redhat.com"
~~~

Now apply the changes to the updated CR.

~~~
$ oc apply -f weather-product-full.yml 
product.capabilities.3scale.net/weather-alerts-api configured
~~~

The complete spec for all the options in the Product CR can handle can be found in the operator documentation here: https://github.com/3scale/3scale-operator/blob/3scale-2.14-stable/doc/product-reference.md