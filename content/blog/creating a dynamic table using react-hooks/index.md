---
title: Creating a dynamic table using bootstrap 4 and react-hooks
date: "2020-03-28T22:12:03.284Z"
description: "How to create a dynamic table using bootstrap 4 and react-hooks"
---

I've been looking at different ways of creating dynamic tables using react, there are different options to do this, but my goal was to have a very easy way to pass a data set and have it displayed without much additional work. One amazing project i run into is [react-admin](https://marmelab.com/react-admin), this is a fully fledged product and you can do many things with it. I recommend to check it out and give it a try. In my case, though, i wanted to do something simple. So i started from scratch.

I will be making use of some common react hooks, if you are not familiar with the basic concepts of react and react-hooks, you can check out [react-hooks](https://reactjs.org/docs/hooks-intro.html). In addition, we will use bootstrap 4.0 to style the table. See [bootstrap](https://getbootstrap.com/) for more information.

### Setup

Let's start by setting up a new react project using [create-react-app:](https://github.com/facebook/create-react-app#creating-an-app)

```react
npx create-react-app react-bootstrap-table
cd react-bootstrap-table
npm start
```

Next, let's hook up bootstrap to define a basic table layout; following the [bootstrap](https://getbootstrap.com/docs/4.4/getting-started/introduction) setup instructions we add the necessary links and scripts to the index.html file.

### Sample data

A simple json list is used, this set will be the data that will be displayed in the table. The list will look like this:

```json
[
  {
    "id": "1",
    "name": "Ghost in The Wires",
    "author": "Kevin Mitnick",
    "released": "08/15/2011"
  },
  {
    "id": "2",
    "name": "Console Wars",
    "author": "Blake J. Harris",
    "released": "05/13/2014"
  },
  {
    "id": "3",
    "name": "The Phoenix Project",
    "author": "Gene Kim, Kevin Behr, George Spafford",
    "released": "12/01/2017"
  }
]
```

To display these records in the page, lets modify the app.js file to load the data from the file.

```react
import React from 'react';
import './App.css';
import db from "./db.json"

function App() {
  return (
    <div className="App">
      { db.map((item, key) =>
        <li key={item.id}>{item.name}</li>
      )}
    </div>
  );
}

export default App;
```

The output should look something like this:

![Alt Text](https://thepracticaldev.s3.amazonaws.com/i/kk2b867kk63ne0hhfqn0.png)

### Functional Components

Next, we create the Table, TableHeader and TableBody function components that will contain our logic and content:

```react
import React from 'react';

const Table = () => {
  return (
    <div>
      <table className="table table-bordered table-hover">
      <TableHeader></TableHeader>
      <TableBody></TableBody>
      </table>
    </div>
  );
}

const TableHeader = () => {
  return(
      <thead className="thead-dark" key="header-1">
          <tr key="header-0">
            <td>Hello i am a table header</td>
          </tr>
      </thead>
  );
}

const TableBody = () => {
  return(
      <tbody>
          <tr key="spinner-0">
              <td>
                 Hello i am a table row
              </td>
          </tr>
      </tbody>
);
}

export default Table;
```

Import the Table component into app.js and add it after the list of items is displayed. You should see the following:

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/2l3xn4ydb89t5a2ax0bb.png)

### Loading data into the components

In order to dynamically add the columns, add a schema.json object that contains an empty record; this basic setup will allows us to define the columns needed to be rendered.

```json
{
  "id": "",
  "name": "",
  "author": "",
  "released": ""
}
```

The data to render the rows and columns is provided to the component as props, and the field names are pulled from the schema object. Lets update the Header component to receive the header fields as props and iterate thru every header field.

```react
const TableHeader = (props) => {
  const { headers } = props;
  return(
    <thead className="thead-dark" key="header-1">
        <tr key="header-0">
          { headers && headers.map((value, index) => {
              return <th key={index}><div>{value}</div></th>
          })}
        </tr>
    </thead>
  );
}
```

The Body component is also updated to receive the header and rows fields and render the rows and columns accordingly.

```react
const TableBody = (props) => {
  const { headers, rows } = props;

  function buildRow(row, headers) {
    return (
         <tr key={row.id}>
         { headers.map((value, index) => {
             return <td key={index}>{row[value]}</td>
          })}
         </tr>
     )
  };

  return(
      <tbody>
        { rows && rows.map((value) => {
                return buildRow(value, headers);
            })}
      </tbody>
);
}
```

And the Table component is updated to receive the header and rows props and pass them to the subcomponents.

```react
const Table = (props) => {
  const { headers, rows } = props;
  return (
    <div>
      <table className="table table-bordered table-hover">
      <TableHeader headers={headers}></TableHeader>
      <TableBody headers={headers} rows={rows}></TableBody>
      </table>
    </div>
  );
}
```

Finally, the app.js file is updated to load the headers from the schema file, pass the headers and rows to the Table component via props, remove the initial loop and add a few styling changes.

```react
import React from 'react';
import './App.css';
import db from "./db.json"
import schema from './schema';
import Table from './Table';

function App() {
  return (
    <div className="container p-2">
      <div className="row">
        <div className="col">
          <Table headers={Object.keys(schema)} rows={db} />
        </div>
      </div>
    </div>
  );
}

export default App;
```

We now see all the header fields from the schema object and its correspondent data being passed to the Table component.

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/73kzs7zy52zgjtbdgdj6.png)

### Loading Data from the server

In many cases, the data to be loaded into our table will come from a server via a REST api call. The component will render the rows only after the data is retrieved from the server. Let's implement a few changes to simulate rendering the data once it is ready and display a spinner while data is not available.

We introduce the useState and useEffect react hooks to simulate loading the data from an api call:

```react
import React, { useState, useEffect }  from 'react';
import './App.css';
import db from "./db.json"
import schema from './schema';
import Table from './Table';

function App() {
  const [ data, setData] = useState(null);

  useEffect(() => {
    new Promise((resolve) => {
        setTimeout(() => {
            resolve(db);
        }, 2000)
    }).then((result) => {
        setData(result);
    })
  });

  return (
    <div className="container p-2">
      <div className="row">
        <div className="col">
          <Table headers={Object.keys(schema)} rows={data} />
        </div>
      </div>
    </div>
  );
}

export default App;
```

And update the TableBody component to show the spinner while data is unavailable:

```react
const TableBody = (props) => {
  const { headers, rows } = props;
  const columns = headers ? headers.length : 0;
  const showSpinner = rows === null;

  function buildRow(row, headers) {
    return (
         <tr key={row.id}>
         { headers.map((value, index) => {
             return <td key={index}>{row[value]}</td>
          })}
         </tr>
     )
  };

  return(
    <tbody>
        {showSpinner &&
          <tr key="spinner-0">
              <td colSpan={columns} className="text-center">
                  <div className="spinner-border" role="status">
                      <span className="sr-only">Loading...</span>
                  </div>
              </td>
          </tr>
          }
        { !showSpinner && rows && rows.map((value) => {
              return buildRow(value, headers);
          })}
    </tbody>
  );
}
```

The table will look like this while loading:

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/g2u6owmnoimtgp5gamtm.png)

### Summary

We now have created a dynamic table using react and bootstrap 4. I will expand the table functionality in later posts.

You can check out a [live demo](https://react-bootstrap-table.alvarobodero.com) of the component and the [source code](https://github.com/abodero/react-bootstrap-table).

See also CodeSandBox: [react-bootstrap-table](https://codesandbox.io/s/react-bootstrap-table-zitkz?fontsize=14&hidenavigation=1&theme=dark)
