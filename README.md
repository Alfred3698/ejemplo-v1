ndex data
You index data using the OpenSearch REST API. Two APIs exist: the index API and the _bulk API.

For situations in which new data arrives incrementally (for example, customer orders from a small business), you might use the index API to add documents individually as they arrive. For situations in which the flow of data is less frequent (for example, weekly updates to a marketing website), you might prefer to generate a file and send it to the _bulk API. For large numbers of documents, lumping requests together and using the _bulk API offers superior performance. If your documents are enormous, however, you might need to index them individually.

Introduction to indexing
Before you can search data, you must index it. Indexing is the method by which search engines organize data for fast retrieval. The resulting structure is called, fittingly, an index.

In OpenSearch, the basic unit of data is a JSON document. Within an index, OpenSearch identifies each document using a unique ID.

A request to the index API looks like this:

PUT <index>/_doc/<id>
{ "A JSON": "document" }
A request to the _bulk API looks a little different, because you specify the index and ID in the bulk data:

POST _bulk
{ "index": { "_index": "<index>", "_id": "<id>" } }
{ "A JSON": "document" }
Bulk data must conform to a specific format, which requires a newline character (\n) at the end of every line, including the last line. This is the basic format:

Action and metadata\n
Optional document\n
Action and metadata\n
Optional document\n
The document is optional, because delete actions don’t require a document. The other actions (index, create, and update) all require a document. If you specifically want the action to fail if the document already exists, use the create action instead of the index action.

To index bulk data using the curl command, navigate to the folder where you have your file saved and run the following command:

curl -H "Content-Type: application/x-ndjson" -POST https://localhost:9200/data/_bulk -u 'admin:admin' --insecure --data-binary "@data.json"
If any one of the actions in the _bulk API fail, OpenSearch continues to execute the other actions. Examine the items array in the response to figure out what went wrong. The entries in the items array are in the same order as the actions specified in the request.

OpenSearch automatically creates an index when you add a document to an index that doesn’t already exist. It also automatically generates an ID if you don’t specify an ID in the request. This simple example automatically creates the movies index, indexes the document, and assigns it a unique ID:

POST movies/_doc
{ "title": "Spirited Away" }
Automatic ID generation has a clear downside: because the indexing request didn’t specify a document ID, you can’t easily update the document at a later time. Also, if you run this request 10 times, OpenSearch indexes this document as 10 different documents with unique IDs. To specify an ID of 1, use the following request (note the use of PUT instead of POST):

PUT movies/_doc/1
{ "title": "Spirited Away" }
Because you must specify an ID, if you run this command 10 times, you still have just one document indexed with the _version field incremented to 10.

Indexes default to one primary shard and one replica. If you want to specify non-default settings, create the index before adding documents:

PUT more-movies
{ "settings": { "number_of_shards": 6, "number_of_replicas": 2 } }
Naming restrictions for indexes
OpenSearch indexes have the following naming restrictions:

All letters must be lowercase.
Index names can’t begin with underscores (_) or hyphens (-).
Index names can’t contain spaces, commas, or the following characters:

:, ", *, +, /, \, |, ?, #, >, or <

Read data
After you index a document, you can retrieve it by sending a GET request to the same endpoint that you used for indexing: