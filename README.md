## audit-logging-framework

An audit logging framework for Relational databases.

### Prerequisite

You must have following softwares already setup-

1. kafka (Pub-Sub model messaging system, a Big Data tool) 
2. zookeeper (It is already shiped with kafka. Kafka uses ZooKeeper to manage the cluster)
3. mongodb (To store audit logs data)
4. node and npm (development and build environment)


### Working

This app has been developed to work with Relational databases. Its working can be explained in following steps-

<dl>
  <dt>To create an audit-log: you hit an API http://localhost:5000/auditLog</dt>
  <dd>
  method: POST

  request:

    {
      tableId: 1,
      rowId: 1,
      oldData: {
        id: 1,
        name: 'varun',
        age: 22,
        profession: 'engineer'
      },
      newData: {
        id: 1,
        name: 'varun kumar',
        profession: 'software developer'
      },
      ipAddress: 'localhost',
      user: {
        email: 'varunon9@gmail.com'
      },
      miscellaneous: {
        platform: 'web',
        os: 'linux'
      }
    }

  Here **oldData** and **newData** are complete data of a row in table. **user** is user details and **miscellaneous** is any other data that you want to store in logs.
  **tableId** and **rowId** are unique identifiers for auditLogs. You can assign all your tables a unique id. 
  This data is pushed to kafka for processing. Processing is basically calculating diff of **oldData** and **newData** and creating a new payload. This new payload is then stored to mongodb. Stored data would look like-

    {
      _id : ObjectId(5ba698f0a5c65c3327f03b67),
      updatedAt: ISODate('2018-09-22T19:34:50.887Z'),
      createdAt: ISODate('2018-09-22T19:33:04.785Z'),
      tableId: 1,
      rowId: 1,
      changeHistory: [
        {
          log: {
            name: 'varun kumar',
            profession: 'software developer'
          },
          date: ISODate('2018-09-22T19:32:54.957Z'),
          _id: ObjectId('5ba698f041c85339413f10e6'),
          ipAddress: 'localhost',
          user: {
            email: 'varunon9@gmail.com'
          },
          miscellaneous: {
            platform: 'web',
            os: 'linux'
          }
        }
      ]
    }

  If you hit `http://localhost:5000/auditLog` again with same tableId and rowId, logs would be pushed to *changeHistory*. When you create a new record in table (in your relational database), you pass **oldData** as `{}` (empty object). In that case all of **newData** would be pushed to *changeHistory*.
  </dd>

  <dt>To retrieve audit-log: you hit same API http://localhost:5000/auditLog</dt>
  <dd>
  method: GET

  queryParams: tableId, rowId, count

  example-

    GET http://localhost:5000/auditLog/?tableId=1&rowId=1&count=5

  In response you would get *changeHistory* of give **tableId** and **rowId** with no of results = *count*
  The data would be sorted according to data i.e. latest 5 results
  </dd>

</dl>

### How to install

1. Clone repo `git clone https://github.com/varunon9/audit-logging-framework.git`
2. Go inside repo `cd audit-logging-framework`
3. Edit `config/index.js` for connection details.
4. Start the app `npm start`
5. Log data from your service class i.e. when data is created/updated in table. POST `localhost:5000/auditLog`
6. Retrieve data when showing audit-logs. GET `localhost:5000/auditLog`
