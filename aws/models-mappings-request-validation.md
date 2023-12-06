# Models, Mappings, Request Validation Notes

### Request Validation
API Gateway can perform basic validation. This enables you, the API developer, to focus on app-specific deep validation in the backend. You can offload basic validation to API Gateway. For the basic validation, API Gateway verifies either or both of the following conditions:

The required request parameters in the URL, query string, and headers of an incoming request are included and non-blank.

The applicable request payload adheres to the configured JSON schema request model of the method.

Validation is performed in the Method Request and Method Response of the API. You can associate models to validate against at the Method Request or Method Response level for each HTTP method. This means that different HTTP methods under a resource can use different models.

### Models
A model in API Gateway allows you to define a schema for validating requests and responses.

The model is used to define the format of the incoming data on the Method Request, or the format of the outgoing data on the Method Response.

### Mappings
Mappings are templates written in Velocity Template Language (VTL) that you can apply to the Integration Request or Integration Response of a REST API. The mapping template allows you to transform data, including injecting hardcoded data, or changing the shape of the data before it passes to the backing service or before sending the response to the client.

You can access information on the payload of the request and other data in your mapping template by using variables.

### Stage Variables
Stage variables are name-value pairs that you can define as configuration attributes associated with a deployment stage of a REST API. They act like environment variables and can be used in your API setup and mapping templates.

With deployment stages in API Gateway, you can manage multiple release stages for each API, such as alpha, beta, and production. Using stage variables you can configure an API deployment stage to interact with different backend endpoints.

You can reference stage variables in Mapping templates through the use of $stageVariables.