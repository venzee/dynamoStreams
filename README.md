# @venzee/dynamo_streams

NodeJs Stream wrappers around DynamoDB requests.

## Installation

`npm i -S @venzee/dynamo_streams`

## Usage

### QueryStream

You can inject DocumentClient instances or DynamoDB instances. The stream uses calls the _query_ method of the injected client with the provided params.
The stream handles pagination and will emit all items found that match the provided query.

See the documentation for [DynamoDB](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/DynamoDB.html#query-property) and [DocumentClient](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/DynamoDB/DocumentClient.html#query-property) for more info;

```js
const client = new AWS.DynamoDB.DocumentClient();
const queryStream = require( '@venzee/dynamo_streams/query' )( client, params );

queryStream
  .on( 'data', doStuffWithData)
  .on( 'error', handleError);
```

### ScanStream

Same behavior as a QueryStream, except we are streaming `scan` results, not `query` results.

## Errors

When _query_ or _scan_ return errors, the 'error' event raised by the stream includes the error raised by the method as well as the parameters used to last invoke the method. This allows client code to retry using custom retry logic, for example.

```js
function runQuery( params, doStuffWithData, onQueryRun ){

  const client = new AWS.DynamoDB.DocumentClient();
  const queryStream = require( '@venzee/dynamo_streams/query' )( client, params );

  queryStream
    .on( 'data', doStuffWithData)
    .on( 'error', onQueryRun )
    .on( 'finish', onQueryRun );
}

function onQueryRun( err ){

  if( err ){

    const { error, queryParameters } = err;

    if( error.code === 'ProvisionedThroughputExceededException' ) return setTimeout( runQuery, 1000, queryParameters, doStuffWithData, onQueryRun );
    return handleError( error );

  }

}
```
