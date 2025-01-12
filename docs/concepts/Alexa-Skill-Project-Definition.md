# Alexa Skill Project Resource Components

Alexa skill is by nature an application which delivers the Voice-User-Interface (VUI) through natural language techniques and cloud infrastructures. Accordingly, `ask-cli` helps Alexa skill developers break down the entire deployment into three steps, represented by three components managed by CLI: **Skill Metadata, Skill Code and Skill Infrastructure**.

Furthermore, those three components are all represented in the `ask-resources.json` file at the root of the skill project. CLI always loads and respects the data from this config file when executing any CLI commands.

## Skill ID
The `SkillID` is the Alexa identifier for the application.


## Skill Metadata
The `SkillMetadata` component stands for all the skill build-time data you need to upload. This includes a great variety of JSON data such as the skill's supported languages, skill's capabilities, in-skill purchases etc. To deploy skill's `SkillMetadata`, CLI directly follows the [Skill Package](https://developer.amazon.com/en-US/docs/alexa/smapi/skill-package-api-reference.html#skill-package-format) structure and calls the service.

Please check the `skillMetadata` field in the JSON example below for its representation in the project config.


## Skill Code
The `SkillCode` component manages the source code for each region. CLI will be in charge of the building process for the code, and passing the code built result to the infrastructure deployer. We support code management for all the regions that [Alexa supports](https://developer.amazon.com/en-US/docs/alexa/custom-skills/host-a-custom-skill-as-an-aws-lambda-function.html#select-the-optimal-region-for-your-aws-lambda-function).

Please check the `code` field in the example below for its representation in project config.


## Skill Infrastructure
The `SkillInfrastructure` represents the configuration on how to deploy skill's code, and tracks the deployment status for continuous deployment. This deployer platform is designed to cope with the variety of serverless frameworks, with necessary interfaces including `userConfig` and `type`. The `deployState` will be tracked after each deploy in the ask-states.json file. For more details, please check [skill infrastructure's deployer](./Deploy-Command.md#Deployer).

Please check the `skillInfrastructure` field in the example below for its representation in project config.

# Project Config For Resources Management
Below shows the example how CLI tracks user's config and the deployment states, using `@ask-cli/cfn-deployer` deployer as an example.

### ask-resources.json (this file is up to configure!)
```jsonc
{
  "profiles": {                             
    "{profileName}": {                      // profile name-specific config

      "skillMetadata": {                    // Alexa skill metadata to deploy
        "src": "./skill-package",           // source folder for skill package (either relative path or absolute path)
        "lastDeployHash": "{hashResult}"    // CLI internal data to optimize the deploy flow
      },

      "code": {                             // Alexa skill code to be built and hosted
        "default": {                        // region for the codebase
          "src": "./code"                   // source folder for codebase
        },
        "{supportedRegion}": {              // supported regions are always in sync with Alexa, which includes default, NA, EU, FE.
          "src": "./code"
        }
      },

      "skillInfrastructure": {              // Alexa skill infrastructure to deploy
        "type": "@ask-cli/cfn-deployer",    // deployer type
        "userConfig": {                     // deployer-specific config
          "awsRegion": "{aws-region}",
          "runtime": "{lambdaRuntime}",
          "handler": "{lambdaHandler}",
          "templatePath": "stack.yaml",
          "targetEndpoint": [               // defaults to api names from skill manifest if not specified
            "alexaForBusiness",
            "custom",
            "flashBriefing",
            "health",
            "householdList",
            "music",
            "smartHome",
            "video"
          ],
          "skillEvents": {                  // skill events support for deployer endpoint(s)
            "publications": [               // list of proactive event names
              "{proactiveEventName1}",
              ...
            ],
            "subscriptions": [              // list of skill/list event names
              "{skillOrListEventName1}",
              ...
            ]
          },
          "artifactsS3": {                  // custom s3 configuration to upload skill build artifact (zip, jar, etc)
            "bucketName": "{bucketName}",   // custom bucket name to store artifacts
            "bucketKey": "{bucketKey.zip}"  // custom bucket object key
          },
          "cfn": {
            "parameters": {                 // additional parameters to pass to the CloudFormation
              "SomeUserParameter1Key": "some value",
              "SomeUserParameter2Key": "another value"
            },
            "capabilities": [               // additional capabilities to pass to the CloudFormation
              "CAPABILITY_NAMED_IAM"        // CAPABILITY_IAM is always passed by default
            ]
          }
        }
      }
    }
  }
}
```

### .ask/ask-states.json (this file is managed by CLI only, should be read-only)
```jsonc
{
  "profiles": {
    "{profileName}": {
      "skillId": "amzn1.ask.skill.xxxxxx",  // skillId is the identifier for the Alexa application

      "skillMetadata": {
        "lastDeployHash": "{hashResult}"    // CLI internal data to optimize the deploy flow
      },

      "code": {
        "default": {
          "lastDeployHash": "{hashResult}"  // CLI internal data to optimize the deploy flow
        },
        "{supportedRegion}": {              // supported regions are always in sync with Alexa, which includes default, NA, EU, FE.
          "lastDeployHash": "{hashResult}"
        }
      },

      "skillInfrastructure": {
        "@ask-cli/cfn-deployer": {          // deployer type
          "deployState": {                  // deployer-specific states
            "default": {
              "s3": {
                "bucket": "{bucket}",
                "key": "{key}",
                "objectVersion": "{version}"
              },
              "outputs": [                  // outputs from the CloudFormation deploy
                {
                  "OutputKey": "{outputKey}",
                  "OutputValue": "{outputValue}",
                  "Description": "{description}"
                }
              ],
              "stackId": "arn:aws:cloudformation:..."
            },
            "{supportedRegion}": { ... }
          }
        }
      }
    }
  }
}
```
