# Tables
## Read and write data with single digit millisecond latency

Durable persistence of structured data is the foundation for all powerful web apps. Data needs to be instantaneous, consistent, secure, and transparently scale to meet demand.

Architect `@tables` defines DynamoDB tables and `@indexes` define global secondary indexes to facilitate more advanced access patterns. 

> Read the official [AWS docs on DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html)

---

- <a href=#local><b>🚜 Work Locally</b></a> 
- <a href=#provision><b>🌾 Provision</b></a> 
- <a href=#sec><b>💰 Security</b></a>
- <a href=#deploy><b>⛵️ Deploy</b></a>
- <a href=#repl><b>🔪 REPL</b></a>
- <a href=#write><b>🔏 Write Data</b></a>
- <a href=#read><b>📖 Read Data</b></a>
- <a href=#stream><b>📚 Stream Data</b></a>

---

<h2 id=local>🚜 Work Locally</h2>

Tables are defined in `.arc` under `@tables` and `@indexes`:

```arc
@app
testapp

@tables
accounts
  accountID *String

cats
  accountID *String
  catID **String

@indexes
accounts
  email *String
```

*Table names are _lowercase alphanumeric_ and can contain _dashes_.* The hash key is indented two spaces and must be of the type `*String` or `*Number`. The optional partition key is defined `**String` or `**Number`.

> **Protip:** table names can be anything but choose a consistent naming scheme within your app namespace; one useful scheme is plural nouns like: `accounts` or `email-invites` 

Running `arc sandbox` will mount the current `.arc` into a local in memory database on `http://localhost:5000`.

---

<h2 id=provision>🌾 Provision</h2>

Running `arc deploy` will setup the following AWS resources:

- `AWS::DynamoDB::Table`

Additionally `AWS::SSM::Parameter` resources are created for every table which can be inspected at runtime:

- **`/[StackName]/tables/[TableName]`** with a value of the generated DynamoDB Table

> All runtime functions have the environment variable `AWS_CLOUDFORMATION` which is the currently deployed CloudFormation stack name; this combined w the runtime `aws-sdk` or `@architect/functions` can be used to lookup these values in SSM

--- 

<h2 id=sec>💰 Security</h2>

By default all runtime functions generated with Architect have one generated <a href=https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#grant-least-privilege target=blank>IAM role</a> with the least privileges possible. This means Lambda functions can only access other resources defined in the same `.arc` file. 

For `@tables` only the following IAM actions are allowed at runtime:

- `dynamodb:BatchGetItem`
- `dynamodb:BatchWriteItem`
- `dynamodb:PutItem`
- `dynamodb:DeleteItem`
- `dynamodb:GetItem`
- `dynamodb:Query`
- `dynamodb:Scan`
- `dynamodb:UpdateItem`
- `dynamodb:GetRecords`
- `dynamodb:GetShardIterator`
- `dynamodb:DescribeStream`
- `dynamodb:ListStreams`

> Note: wider account access can be explicitly granted with custom resource policies

---

<h2 id=deploy>⛵️ Deploy</h2>

- `arc deploy` to deploy with CloudFormation to staging
- `arc deploy production` to run a full CloudFormation production deployment

---

<h2 id=repl>🔪 REPL</h2>

- `arc repl` to connect to a local in memory sandbox
- `arc repl staging` to connect to staging tables
- `arc repl production` to connect to production tables 

---

<h2 id=write>🔏 Write Data</h2>

`put` with Node

```javascript
let arc = require('@architect/functions')

let data = await arc.tables()
await data.cats.put({fluffID: 2, name: 'sutr0'})
```

`update` with Ruby

```ruby
require 'architect/functions'

cats = Arc::Tables.table tablename: 'cats'

cats.update({
  key: {
    fluffID: 2
  },
  update_expression: 'set colour = :colour',
  expression_attribute_values: {
    ':colour' => 'grey'
  }
})
```
 
And `delete` with Python

```python
import arc.tables

cats = arc.tables.table(tablename='cats')
cats.delete_item(Key=1)
```

---

<h2 id=read>📖 Read Data</h2>

`scan` with Node

```javascript
let arc = require('@architect/functions')

let data = await arc.tables()
let cats = await data.cats.scan({})
```

`query` with Ruby

```ruby
require 'architect/functions'

cats = Arc::Tables.table tablename: 'cats'
result = cats.query {key_condition_expression: 'fluffID = 1'}
```

`get` with Python

```python
import arc.tables

cats = arc.tables.table(tablename='cats')
cat = cats.get_item(Key=1)
```

---

<h2 id=stream>📚 Stream Data</h2>

Stream changes on a DynamoDB table to a Lambda function.

```arc
@app
testapp

@tables
cats
  catID *String
  stream true
```

> `arc init` creates `src/tables/cats` local code and 
> `arc deploy` to publishes to Lambda

<section class="code-examples">

Node
```javascript
exports.handler = async function stream(event) {
  console.log(event)
  return true
}
```

Ruby
```ruby
def handler(event)
  puts event
  true
end
```

Python
```python
def handler(event, context):
    print(event)
    return True
```

</section>

---

## Resources

- [`DynamoDB`](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/DynamoDB.html)
- [`DynamoDB.DocumentClient`](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/DynamoDB/DocumentClient.html)
- [Best Practices](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/best-practices.html)

---
