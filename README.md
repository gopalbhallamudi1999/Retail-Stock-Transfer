# Retail Stock Transfer Management

An SAP Cloud Application Programming Model (CAP) demo for moving stock between retail stores. Hyderabad requests stock from Bengaluru, an inventory controller approves it, and the two stores confirm dispatch and receipt.

## Run locally

Install Node.js 22 or 24 and npm, then run:

```sh
npm ci
npm run watch
```

Open [the Fiori app](http://localhost:4004/transfers/index.html). SAPUI5 loads from the SAP CDN. Sign in with a demo identity when prompted. If the browser does not prompt from the app, open [the service metadata](http://localhost:4004/odata/v4/stock-transfers/$metadata) first. Use separate browser sessions or curl credentials to switch users because browsers cache HTTP Basic credentials.

| User | Password | Role |
| --- | --- | --- |
| hyderabad | demo | Destination store manager. Requests and receives stock. |
| controller | demo | Inventory controller. Approves transfers. |
| bengaluru | demo | Source store manager. Confirms dispatch. |

Demo data is loaded into in-memory SQLite and resets each time the server restarts.

## Demo workflow

1. As **hyderabad**, choose **Request transfer**. Select Bengaluru as the source, Hyderabad as the destination, and Wireless Headphones as the product. Enter a quantity of 10.
2. As **controller**, open the transfer and choose **Approve**. Bengaluru still has 50 units on hand, with 10 reserved.
3. As **bengaluru**, choose **Dispatch**. Bengaluru's balance falls to 40 and the transfer moves to **InTransit**.
4. As **hyderabad**, choose **Confirm receipt**. Hyderabad's balance rises from 5 to 15 and the transfer moves to **Received**.
5. Request another 45 units. CAP blocks approval because Bengaluru has 40 available.

Run `npm test` to verify the same workflow over OData, including role restrictions, stock rollback on failed approval, audit fields, and duplicate receipt prevention.

## Build and deploy to SAP BTP

Run `npm run build:mta` from the project root. It uses the Cloud MTA Build Tool and requires GNU Make. The ReleaseOwl pipeline can collect `mta_archives/retail-stock-transfer-cap_1.0.0.mtar` as its build artifact.

For deployment, target a Cloud Foundry org and space with SAP HANA Cloud and XSUAA entitlements. Install the Cloud Foundry CLI MultiApps plugin and run:

```sh
cf deploy mta_archives/retail-stock-transfer-cap_1.0.0.mtar
```

Open the app router URL from the deployed application routes. Assign the generated inventory controller role collection. In the SAP BTP cockpit, create a role from the **StoreManager** role template for each store, set the **StoreID** attribute to its store UUID, and assign it through a role collection. Use the UUIDs in `test/data/retail-Stores.csv` if reproducing the local demo.

The database layer deploys with empty tables. A database administrator can load demo stores, products, and inventory from the CSV files in `test/data/` into a dedicated demo HDI container, in that order. Keep operational data provisioning separate from MTAR deployment so application upgrades preserve inventory.

## Project layout

| Path | Contents |
| --- | --- |
| `db/` | CDS data model |
| `srv/` | OData V4 service, role checks, and inventory transactions |
| `app/` | Fiori elements List Report, Object Page, and CDS UI annotations |
| `test/` | SQLite fixtures and API workflow test |
| `mta.yaml` | Cloud Foundry app router, CAP service, HANA deployer, and XSUAA configuration |
