# CloudFormation - infrastructure as code

_Lots of questions but at a high level - the foundation of EBS_

**Infrastructure as code**

- too much manual work up untill now. Hard to reproduce, takes a lot of time
- turn infrastructure into code

CloudFormation is a declarative way to outline AWS infrastructure for your resources

- Much more reproductible
- Version controllable
- Changes can pass through code review
- Free, you pay for the resources
- automate your infrastructure
- generate diagrams automatically
- declarative programming (no need to figure out order)
- Separation of concerns, isolate your stacks infrastructures
  - VPC stacks
  - Network stacks
  - App stacks

#### CloudFormation how it works?

Templates go in S3, to update a template upload a new one, like migrations, CF runs the diff and updates. _Deleting a stack cleans everything_

- Edit templates in yaml
- Use the CLI to deploy templates

### CloudFormation Templates components (one course section for each):

1. **Resources**: your AWS resources declared in the template (MANDATORY)
2. **Parameters**: the dynamic inputs for your template
3. **Mappings**: the static variables for your template
4. **Outputs**: References to what has been created
5. **Conditionals**: List of conditions to perform resource creation
6. **Metadata**

_Templates helpers: _

1. References
2. Functions

#### Resources

The core of a CF template (MANDATORY), represent the different aws billables, can be declared and reference each other
CF manages creating, updating and deleting resources (>= 224 types of resources). Almost all services are supported, lambda custom resources can fill in the niches

Identifiers come in the form:

```
AWS::aws-product-name::data-type-name
```

The template can be visualized in a tool
_Attention: you can't create dynamic amounts of resources, EVERYTHING IS DECLARED, no code generation here_

#### Parameters

- Provide input for the CF template
- You want to **Reuse** the templates
- some inputs have to be determine JIT
- Use it when the configuration is prone to change in the future, prevents template re-uploads (and the redeploys associated)

Parameters have various possible constraints, some for string validation, some for typing

- Type:
  - String
  - Number
  - CommaDelimitedList
  - `List<Type>`
  - AWS Parameter (to help catch invalid values – match against existing values in the AWS Account)
- Description - Constraints
- ConstraintDescription (String)
- Min/MaxLength
- Min/MaxValue
- Defaults
- AllowedValues (array)
- AllowedPattern (regexp)
- NoEcho (Boolean) (for secrets)

Use the parametters with the function `Fm::Ref` or shorthand `1Ref`

_Pseudo parameters_ parameters that AWS automatically injects:

- AccountId
- Region
- StackId, etc

#### Mappings

Fixed variables within the template ( its a map ), handy for simple substitutions, environment variables.

- Good for stuff you can know in advance:
  - region
  - Az
  - account

if you don't know what it may be use a parameter

```
Fn::FindInMap: !FindInMap [MapName, TopLevelKey, SecondLevelKey]
```

#### Outputs (popular exam question)

- If you export outputs they can be used into other stacks
- The Cli can view the outputs in the console
- Usefull to find your ids
- Best way to collaborate cross-stack
- the Export block contains the name that has to be referenced to import
- use the ImportValue function
- you cant delete the underlying stack without deleting the references

```
ImportValue: SSHSecurityGroup
```

#### Conditions (if-else)

- based on whatever, and can reference other components
- good for environments

```
!Equals [ !Ref EnvType, prod]
!And
!If
!Not
!Or
```

- Conditions can be applied as values to any other component, resource/output, etc

#### Intrinsic Functions:

Fn:

- Ref:
  - On a parameter returns value
  - on a resource returns the physical ID
- GetAtt:
  - get attributes, to list all view the docs
- FindInMap: (map name, top level key, second level key)
- ImportValue: imports outputs from other templates
- Join: `Join [delimiter, <comma-dlimited list of values>]`
- Sub: basic string replace

### CloudFormation Rollbacks

- On stack creation fail:

  - Default: everything rolls back (gets deleted), check logs
  - you can prevent the default

- On Update Fails:
  - Auto rollback to the previous working state

### CloudFormation Stack Notifications

- To Send Events to an SNS topic, enable SNS integration using Stack Options
- The SNS can then call a lambda to trigger whatever behavior you wish

#### Change Stets

- You need to know what s going to change
- Can't tell you if it's going to be successfull though

#### Nested Stacks

- To update a nested stack you have to update the parent

| cross                                                   | Nested                                               |
| ------------------------------------------------------- | ---------------------------------------------------- |
| different lifecycles                                    | component reuse, like using the same config on a ALB |
| use exports and imports                                 | the nested one only matters to the parent            |
| when you pass values to many stacks                     | focused on reuse, more like inheritance              |
| focused on configuration sharing, more like composition |                                                      |

#### Stack Sets

- One operation: multiple accounts and regions updated
- Only Administrators can create StackSets
  - trusted accounts can create update and delete stacks from the StackSets
- When you update a StackSet **ALL** stack instances are updated in **ALL CCOUNTS AND REGIONS**

#### CloudFormation Drift

- CF doesn't protect against manual change, that's drift
- `CF drift` is a tool to detect that.

#### CloudFormation Stack Policies

By default all update actions are allowed on all resources in an update

A stack policy is a JSON that defines what update actions are allowed on specific resources

- protects from unintentional updates (but not from drift)
- when you set one the default shifts to all protected
- you specify all the allow on the resources
