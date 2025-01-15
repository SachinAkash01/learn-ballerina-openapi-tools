# Ballerina OpenAPI Tool

## OpenAPI to Ballerina

### Client Generation

Client with payload binding `--mode client` only for success response. We need to set the `--mode` into client in order to generate the client from the given OpenAPI specification.
	E.g: `bal openapi -i <openapi-contract> --mode client`

In the Ballerina OpenAPI tool, the default client won’t bind any error status code while generating the client. So for that, we can give the option `--status-code-binding` when Cgenerating the client in order to add the status code binding for the client generation.
	E.g: `bal openapi -i <openapi-contract> --status-code-binding --mode client`

There are scenarios where the OpenAPI spec has examples for the client under the response section. So before actually generating the client, we can test only these examples by generating them as a mock client. In order to do this, we can use the `--mock` command option and generate the mock client and check the functionalities and the user behavior while they develop their system. Then they can replace this mock client with the actual client.
	E.g: `bal openapi -i <openapi-contract> --mock client`

### Service Generation

We can generate the service declaration from the given OpenAPI contract by using the `--mode` command option as we previously did in the client generation.
	E.g: `bal openapi -i <openapi-contract> --mode service`

OpenAPI specification is a way to describe the service interface of REST APIs. The counterpart of this in Ballerina is service object types. This `--with-service-contract` command option is to enable the code generation for the Ballerina service contract object type. This service contract type is specific to the OpenAPI contract we are providing.
	E.g: `bal openapi -i store.yaml --mode service --with-service-contract`
The above command generates a Ballerina service in a file named `service_contract.bal` and relevant schemas in a file named `types.bal` for the `store.yaml` OpenAPI contract. The above command can be run from anywhere on the execution path. It is not mandatory to run it from within a Ballerina package.

We have another command option called `--with-serivce-type` which  we can use to generate the basic http service type from a given OpenAPI contract file.
	E.g: `bal openapi -i <openapi-contract> --mode service --with-service-type`
And then we have this command option called `--without-data-binding` which is a proxy generator. Here we don’t generate any response or request data binding. It is just a skeleton with some user parameters.
	E.g: `bal openapi -i <openapi-contract> --mode service --with-service-type`

### [OAS to Ballerina] Common Command Options

We have this command option called `--single-file` which can be used to generate the Ballerina service or client with related types and utility functions in a single file. When this command is specified, it combines all the generated components (e.g: service/client definitions, types and utility functions) into a single file instead of generating them in a separate files.
	E.g: `bal openapi -i <openapi-contract> --mode client --single-file`

We have this `add` subcommand where we can automatically update the `Ballerina.toml` file with OpenAPI tool configurations. Adding these configurations to the `Ballerina.toml` file will generate a client during the package build. This eliminates the need to commit the generated code.
Executing the following OpenAPI `add` sub-command, along with the OpenAPI to Ballerina CLI options, will update the `Ballerina.tom`l with OpenAPI tool configurations. It is mandatory to provide the `id` and `filePath` attributes for the configurations. The other attributes are optional.
	E.g: `bal openapi add -i <openapi-contract> --id <client_id>`

In order to generate the Ballerina service stub with a subset of tags defined in an OpenAPI contract, we can use the `--tags` option and specify the tags we need as specified in the OpenAPI definition.
E.g: `bal openapi -i <openapi-contract> [--tags <”tag1”, “tag2”]` 

To generate the Ballerina client or service stub with a subset of operations defined in the OpenAPI contract, use the `--operations` option and specify the operations you need as specified in the OpenAPI definition.
	E.g: `bal openapi -i <openapi-contract> [--operations <”op1”, “op2”>]`

If your OpenAPI contract includes JSON schema properties that are not marked as `nullable:true`, they might return as null in some responses. It results in a JSON schema to Ballerina record data binding error. If you suspect this can happen for any property, it is safe to generate all data types in the generated record with Ballerina nil support by turning on this flag.
	E.g: `bal openapi -i <openapi-contract> --nullable`

We have this `--use-sanitized-oas` command option which enables service/client code generation by modifying the given OAS to follow Ballerina language best practices (such as naming conventions).
	E.g: `bal openapi -i <openapi-contract> --use-sanitized-oas`
We have introduced a new sub command called `align` to carry out this exact feature in a more user friendly way. We will discuss it in the upcoming topics.

## Ballerina to OpenAPI

Currently we support both Ballerina service contract type and the service implementation to generate OpenAPI specification. If the user has provided only the http service type, we won’t be able to generate the OpenAPI specification.

### Constraint Mapping

Constraints are the OpenAPI schema validations equivalent in the Ballerina programming language. The Ballerina OpenAPI Tool has support for mapping these constraints in Ballerina to OAS by using the constraint mapper of the module.

### Interceptor Mapping

We can successfully map the Ballerina HTTP Interceptors in the generated OpenAPI specification by using this Interceptor mapper in the Ballerina OpenAPI tool.

### HATEOAS Mapping

In Ballerina we can generate links between the resources by using the `ResourceConfig` annotation in Ballerina. These links (Hypermedia constraints or HATEOAS links) can be reflected in the generated OpenAPI specification using the HATEOAS mapper in the Ballerina OpenAPI tool.

### Example Mapping

If a user has provided examples in the Ballerina code by using the `@openapi:Example` annotation, those examples will be successfully mapped into generated OpenAPI specification by using the Ballerina OpenAPI Example Mapper.

### Service Meta Info Mapping

We have Ballerina `ServiceInfo` annotation to bind all the meta info such as service contract details, URL, etc. These details will be mapped into the OpenAPI info section in the generated OpenAPI specification by using the Service Meta Info mapper.

### Cache Annotating Mapping

There is also a cache annotation mapping supported by the Ballerina OpenAPI tool which is coming from the HTTP side as well.

## OpenAPI to OpenAPI

### Align OpenAPI Contract

By using the `bal openapi align` command, we can give an OpenAPI specification as input, modify it to align with Ballerina’s best practices (e.g: adhering to naming conventions), and output a modified OpenAPI specification. The output file will be named as `aligned_ballerina_openapi.json`. Then we can use this aligned OpenAPI specification to generate our client or service stubs and use them without having to sanitize them with Ballerina’s best practices after generating them.

### Flatten OpenAPI Contract

The 'bal openapi flatten` command will move all the inline schemas in the OpenAPI specification into the components section in OAS with their name. In the `flatten` subcommand what we generate is some names for the inline schema. We move the inline schemas to the component section and provide a name for that and we refer to that in the place where we have the inline schema. We get these names from the swagger parser, but that is not compliant with our Ballerina naming conventions. So here what we do is, we get the name from the swagger parser and we sanitize the newly added names. So when we flatten, we will get new names for the inline schemas, and these new names will be Ballerina friendly.

The new names we are getting from the swagger parser are not Ballerina friendly, that’s why we are sanitizing them into a Ballerina friendly way with this `flatten` subcommand.

In the OAS to OAS methods, our normal procedure is to do `bal openapi flatten` first and then do the `bal openapi align` to the flattened OpenAPI contract. After that we can use the aligned OpenAPI contract to generate client/service implementations.

# OpenAPI Tool Usage

## OpenAPI to Ballerina

### Client Generation

When it comes to the client generation, it behaves like a CLI command. We can use it to generate connectors from the OpenAPI specification. And also we can integrate this into the `bal build` command by adding OpenAPI tool configurations to the `Ballerina.toml` file. In this case, once the `bal build` command is executed, the Ballerina client code will be generated according to the provided tool configurations in the toml file.

### Service Generation

In service generation also, we have a CLI command for the service generation. Other than that we have another HTTP mediator proxy generator which is required by the Choreo team. We are planning to integrate these features to the `bal build` command as we did in the client generation. This is a BI team requirement.

## Ballerina to OpenAPI

Ballerina to OAS also has the CLI command integration. It has the http modules for embedding OAS file for service as an introspection resource (an inbuilt plugin) which behaves as an inbuilt compiler plugin. When we are running the service with openapi embedded=true, it will generate the OpenAPI specification for the given service declaration node and attach it to the section in the http `@openapi` annotation (openapi serviceinfo embed true).

Then we have this `bal build --export-openapi` analyser compiler plugin command. When we run this command it will hit every service declaration node in the given package and generate the particular OpenAPI specification and add it under the `target` directory in the openapi folder. This is also a requirement from the Choreo side.

So the other usage is VSCode `Try it` code lens swagger UI which is a LS extension (Language Server). When we write Ballerina code in VS Code, we can see the `Try it` pop up above the service. When we go to this `Try it` section, we can see the rendered Swagger UI there.. So this is done via this language extension.

There’s another feature called external OAS generation service catalog which is currently being implemented and yet not released to the public.

## `import ballerina/openapi` Ballerina Module

There are two major reasons why we use this Ballerina OpenAPI importers,

The First one is validator which is a separate jar but currently we are deprecating it which means we don’t carry on any maintenance under the validator since we have service contract type generator along with the service generation. The reason behind this is, if we have the service contract type, it can validate against your service declaration node by defining the OpenAPI operations. But still we don’t remove this from the openapi-tools repo.

The other main reason why we are having these `ballerina/openapi importers is to add Examples, ServiceInfo, and ResourceInfo annotations. These annotations are implemented under the `ballerina/openapi` module.

## OpenAPI to OpenAPI

As we have previously mentioned, we have these newly introduced CLI commands called `align` and `flatten` and those are the current main usages of OAS to OAS openapi CLI.

And there’s another feature where we have planned to add it in the future, which is to integrate the logic of OAS to Ballerina code generation.
