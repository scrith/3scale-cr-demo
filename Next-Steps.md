# Next Steps

Now that our objects are created we will need to create the first developer application and test the API.

In environments with integrated SSO, the OCP CR methodology cannot be used to create Applications because the Application CR spec must reference a DeveloperUser CR object. 

So the next steps will need to take place inside the 3scale admin interface.

## Create Developer Application

1. Access the Product API page and goto Applications > Listing.
1. Click the 'Create Application' button.
1. Fill in the necessary details

This process creates a user_key that will be needed for testing.