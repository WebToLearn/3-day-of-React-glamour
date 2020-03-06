# Day 2 - Exercises

## Table of contents

## Exercise 0 - Configuration

- [Exercise 0 - Configuration](#exercise-0---configuration)
  - [Download all npm dependencies](#download-all-npm-dependencies)
  - [Install and use JSON Server](#install-and-use-json-server)
  - [Add global styles](#add-global-styles)
- [Exercise 1 - Create blotter-view, fx-rates-view and widget components](#exercise-1---create-blotter-view-fx-rates-view-and-widget-components)
- [Exercise 2 - Blotter View section](#exercise-2---blotter-view-section)
  - [Transaction model](#transaction-model)
  - [Implement polling mechanism](#implement-polling-mechanism)
  - [Blotter View render](#blotter-view-render)

### Download all npm dependencies

- go to _Day-2\Exercise\Code\fx-trading-app_:

```bash
cd 3-Days-of-React-glamour\Day-2\Exercise\Code\fx-trading-app
```

- run _npm install_ to download all dependencies:

```bash
npm install
```

### Install and use JSON Server

- because we do not have a backend server and a link to a real database at this moment, we will simulate having some data using _JSON Server_
- the first step is to install it (globally), using the following command:

```bash
npm install json-server -g
```

- next, we have to start it with the 2 existing files - containing _quote_ and _trade_ data
- make sure you are in the following path, where both JSON files are situated: _Day-2/Exercise/Code/fx-trading-app_
- run these commands in separate terminal windows:

```bash
json-server --watch db.trade.json --port 8210
json-server --watch db.quote.json --port 8220
```

### Add global styles

- add the following styles to _index.css_ file:

```CSS
@import-normalize;

html, body{
   height: 100%;
   color: #373A3C;
}

h1,h2,h3,h4,h5,h6 {
   color: #7C7C7C;
}

.btn-primary {
   background-color: #3496F0;
}

.btn-link {
   color: #3496F0;
}
.table-striped tbody tr:nth-of-type(odd) {
   background-color: #F2F2F2;
}

.flex {
   display: flex;
}

.flex-vertical-centered {
   display: flex;
   align-items: center;
}

.title {
   margin-bottom: 30px;
   padding-bottom: 20px;
}

.title-border {
   border-bottom: 1px solid #DDDDDD;
}

.screen-full-height {
   height: 100vh;
}
```

## Exercise 1 - Create blotter-view, fx-rates-view and widget components

- from design mockup, we can see that it can be divided in 2 big sections: **FX Rates View** and **Blotter View**. Also, the first one contains many widgets looking the same, so this can be also splitted into **Widget** components
- so, _dashboard-page_ component will use 3 smaller components which will need to be created:
  - _blotter-view_
  - _fx-rates-view_
  - _widget_

![Components](https://raw.githubusercontent.com/WebToLearn/3-day-of-React-glamour/master/Design/img/components.png "Components")

- create _components_ folder:

```bash
cd src
mkdir components
cd components
```

- create these 3 new React components, using the example below, as children of _components_ folder:
  - blotter-view.tsx
  - fx-rates-view.tsx
  - widget.tsx

```JavScript
import React, { Component } from 'react'

interface Props {
}
interface State {
}

class BlotterView extends Component < Props, State > {
    state = {}

    render() {
        return (
            <div>
                blotter view
            </div>
        )
    }
}

export default BlotterView
```

- in the design, we can see on the top of the dashboard a navbar, containing the logo and _Logout_ button
- next step is to include into _dashboard-page_ component the navbar, _blotter-view_ and _fx-rates-view_ and also use _bootstrap_ to place them
- so, in _dashboard-page.ts_ we will have:

```HTML
import React from 'react';
import FxRatesView from '../components/fx-rates-view';
import BlotterView from '../components/blotter-view';
import '../styles/dashboard-page.css';

function DashboardPage() {
  return (
    <div>
      <header>
        <nav className="navbar">
          <img className="fx-main-logo" alt="fx-main-logo" src="./img/logo-main.svg" />
          <button className="btn btn-logout">Log out</button>
        </nav>
      </header>

      <main className="dashboard-container">
        <div className="row">
          <div className="col-sm">
            <FxRatesView></FxRatesView>
          </div>
          <div className="col-sm">
            <BlotterView></BlotterView>
          </div>
        </div>
      </main>
    </div>
  );
}

export default DashboardPage;
```

- create _styles_ folder so we can add dashboard styles:

```bash
cd ..
mkdir styles
cd styles
```

- create _dashboard-page.css_ and add the following styles:

```CSS
.dashboard-container {
    padding: 2rem 3rem;
}

.navbar {
    padding: 0.5rem 3rem;
    border: 1px solid #DDDDDD;
}

.fx-main-logo {
    width: 70px;
    height: 50px;
}

.btn-logout {
    border: 1px solid #dddddd;
    color: #7c7c7c;
}

.btn-logout:hover {
    background-color: #F2F2F2;
    opacity: 0.8;
}
```

## Exercise 2 - Blotter View section

### Transaction model

- create a new folder named _models_ into _Day-2\Exercise\Code\fx-trading-app\src_ which will contain all required models:

```bash
cd Day-2\Exercise\Code\fx-trading-app\src
mkdir models
cd models
```

- let's make a new file there, named _transaction.tsx_,representing the _Transaction_ model:

```JavaScript
export interface Transaction {
    id?: number;
    username: string;
    primaryCcy: string
    secondaryCcy: string;
    rate: number;
    action: string;
    notional: number;
    tenor: string;
    date: number;
    ccyPair?: string
}
```

### Implement polling mechanism

- we want to simulate the real-time behavior for getting the transactions. This is the reason why we implement polling mechanism and using fetch API:

```Javascript
class BlotterView extends Component<Props, State> {
    state = {
        timer: null,
        transactions: [],
    }
    timer = null;
    componentDidMount() {
        this.setState({ timer: window.setInterval(() => this.getTransactions(), 2000) });
    }
    componentWillUnmount() {
        this.setState({ timer: null });
    }

    getTransactions() {
        fetch("http://localhost:8210/transactions")
            .then(response => response.json())
            .then(transactions => {
                // Add currency pairs to transactions
                const transactionsWithCcyPair: Transaction[] = transactions.map((transaction: Transaction) => ({ ...transaction, ccyPair: `${transaction.primaryCcy}/${transaction.secondaryCcy}` }));
                this.setState({ transactions: transactionsWithCcyPair });
            });
    }

    render() {
        return (
            <div>
                blotter view
            </div >
        )
    }
}
```

- so, we have the following behavior:

  - when we initiate this component (_componentDidMount_), we should call polling mechanism, which calls _getTransactions()_
  - _componentWillUnmount_ unsubscribes from getting transactions

### Blotter View render

- now is the time to display the table:

```Javascript
...
render() {
    const transactionsList = this.state.transactions.map((trade: Transaction) =>
        <tr key={trade.username + trade.date}>
            <td>{trade.id}</td>
            <td>{trade.username}</td>
            <td>{trade.ccyPair}</td>
            <td>{trade.rate}</td>
            <td>{trade.action}</td>
            <td>{trade.notional}</td>
            <td>{trade.tenor}</td>
            <td>{trade.date}</td>
        </tr>
    )
    return (
        <div>
            <div className="title title-border">
                <h4>Blotter View</h4>
            </div>
            <div className="table-responsive">
                <table className="table table-striped table-sm">
                    <thead className="blotter-table-header">
                        <tr>
                            <th>ID</th>
                            <th>
                                <span>Username&nbsp;</span>
                            </th>
                            <th>
                                <span>Ccy Pair&nbsp;</span>
                            </th>
                            <th>Rate</th>
                            <th>
                                <span>Action&nbsp;</span>
                            </th>
                            <th>
                                <span>Notional&nbsp;</span>
                            </th>
                            <th>Tenor</th>
                            <th>
                                <span>Transaction Date&nbsp;</span>
                            </th>
                        </tr>
                    </thead>
                    <tbody>
                        {transactionsList}
                    </tbody>
                </table>
            </div>
        </div >
    )
}
```

- here we display a table containing the following information from all transactions got from the backend:

  - id
  - username
  - ccyPair
  - rate
  - action
  - notional
  - tenor
  - date

- we need also to create _blotter-view.css_ and add the table styles:

```CSS
.blotter-table-header {
    background: #3496F0;
    color: white;
    overflow: hidden;
    white-space: nowrap;
}
```

- dont forget to import _blotter-view.css_ into _blotter-view.ts_

```Javascript
import './../styles/blotter-view.css'
```

## Exercise 3 - FX Rates View section

### Rate model

- into _Day-2/Exercise/Code/fx-trading-app/src/app/models_, let's make a new file, named _rate.tsx_, representing the _Rate_ model:

```JavaScript
export interface Rate {
    buyRate: number;
    sellRate: number;
    ts: number;
}
```

### Widget model

- in the same location, _Day-2/Exercise/Code/ui/src/app/models_, we want to have the _widget_ class (a new file is required, _widget.tsx_):

```JavaScript
export class Widget {
  constructor(
    public primaryCcy: string,
    public secondaryCcy: string,
    public buyRate: number,
    public sellRate: number,
    public notional: number,
    public tenor: string,
    public pickCCYState: boolean
  ) {}
}
```

### Widget component