---
author_name: Thomas Jung
author_profile: https://github.com/jung-thomas
auto_validation: true
time: 15
tags: [ tutorial>beginner, software-product-function>sap-cloud-application-programming-model, products>sap-business-application-studio, products>sap-hana]
primary_tag: products>sap-hana-cloud
parser: v2
---

# Create Database Artifacts Using Core Data Services (CDS) for SAP HANA Cloud

<!-- description --> Use SAP Cloud Application Programming Model (CAP) and Core Data Services (CDS) to generate SAP HANA Cloud basic database artifacts.

## You will learn

- How to use SAP Cloud Application Programming Model (CAP) Core Data Services (CDS) to create simple database entities
- How to define database-agnostic artifacts in the persistence module

## Prerequisites

- This tutorial is designed for SAP HANA Cloud. It is not designed for SAP HANA on premise or SAP HANA, express edition
- You have created an project according the previous tutorial -- see [SAP HANA Cloud, Create an SAP Cloud Application Programming Model project](hana-cloud-cap-create-project).

## Video Version

Video version of tutorial:

<iframe width="560" height="315" src="https://www.youtube.com/embed/uS_vT-gHYMo" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

### Create database entities

The SAP Cloud Application Programming model utilizes core data services to define artifacts in the database module. Because this model is meant to be database-agnostic -- i.e., work with any database -- it does not allow you to leverage features that are specific to SAP HANA Cloud. For this reason, you will also create two tables that do not require any advanced data types.

1. In the `db` folder, right mouse click and choose **New File**

    ![New File](new_file.png)

1. Use the following name:

    ```Name
    interactions.cds
    ```

    ![Interactions.cds](interactions_cds.png)

1. Use the following content in this new file:

    ```CAP CDS
    namespace app.interactions;

    using {
        Country,
        Currency,
        cuid,
        managed
    } from '@sap/cds/common';

    type BusinessKey : String(10);
    type Price       : Decimal(10, 2);
    type Text        : String(1024);

    entity Headers : cuid, managed {
        items   : Composition of many Items
                      on items.interaction = $self;
        partner : BusinessKey;
        country : Country;
    };

    entity Items : cuid {
        interaction : Association to Headers;
        text        : localized Text;
        date        : DateTime;
        @Semantics.amount.currencyCode: 'currency'
        price       : Price;
        currency    : Currency;
    };
    ```

    > What is going on?
    >
    > You are declaring two entities with relationships between each other. The design-time artifacts declared in this file will be converted to run-time, physical artifacts in the database. In this example, the entities will become tables.

1. We are using a reusable set of content (lists of countries, currencies, etc) provided by SAP in the above model. We also have to add this dependency to our project. From the command line issue the following command to do so:

    ```shell
    npm add @sap/cds-common-content --save
    ```

### Create service interface

1. In the `srv` (**not `src`!**) folder create another file and name it `interaction_srv.cds`

    ```Name
    interaction_srv.cds
    ```

    ![interaction_srv.cds](interactions_srv.png)

1. Use the following content in this new file:

    ```CAP CDS
    using app.interactions from '../db/interactions';
    using {sap} from '@sap/cds-common-content';

    service CatalogService {

        @odata.draft.enabled: true
        entity Interactions_Header as projection on interactions.Headers;

        entity Interactions_Items  as projection on interactions.Items;

        @readonly
        entity Languages           as projection on sap.common.Languages;
    }
    ```

1. Save all.

    > What is going on?
    >
    > You are declaring services to expose the database entities you declared in the previous step.

1. From the terminal issue the command: `cds build --production`

    ```shell
    cds build --production
    ```

    ![cds build](cds_build.png)

1. Look into the console to see the progress. You can scroll up and see what has been built

    ![cds build log](cds_build_log.png)

### Explore generated design-time artifacts

1. If you pay attention to the build log in the console, you will see the `CDS` artifacts were converted to `hdbtable` and `hdbview` artifacts. You will find those artifacts in a new folder under `/gen/db/src` called `gen`.

    ![Results of cds build](generated_objects.png)

1. You will now deploy those objects into the HANA Database creating tables and views. We will use the SAP HANA Projects view to do this. Please expand this view and you will see the following:

    ![SAP HANA Projects view](hana_projects_view.png)

1. We need to bind our project to a Database Connection and HDI container instance. Press the **bind** icon to being the process.

    ![Bind Project](start_bind.png)

1. If you receive either of these two following warning dialogs, please just choose **Continue** (nothing will be deleted because we are create a new HDI Container Instance) and **Enable** (automatic undeploy is quite helpful during the development process for the reasons described in this dialog)

    ![Binding Warning #1](warning1.png)

    ![Binding Warning #2](warning2.png)

1. The bind process will start a wizard where you will be prompted for values via the command pallet at the top of the SAP Business Application Studio screen. You might be asked to confirm your Cloud Foundry endpoint and credentials depending upon how long it has been since you last login.

    ![Confirm Credentials](confirm_credentials.png)

1. Your first choice will be for the binding option.  Choose `Bind to an HDI container`.

    ![Bind to an HDI container](bind_hdi.png)

1. You might be presented with options for existing service instances (if you've completed other tutorials or have performed other HANA development). But for this exercise we want to choose **Create a new service instance**

    ![Create a new service instance](create_Service_instance.png)

1. To make subsequent steps easier, shorten the generated name to `MyHANAApp-dev` + a group number or your initials if you are doing this tutorial as part of a group workshop/shared environment. This makes sure that everything remains unique per participant. Remember the value you used here and adjust the name in the subsequent steps. The remaining screenshots will always just show the base name.

    ![Generated Service Name](generated_service_name.png)

1. It will take a minute or two for the service to be created in HANA. A progress bar will be shown in the message dialog

    ![Service Creation Progress](progress.png)

1. Sometimes the binding step fails due to a timing issue. If so simply repeat the binding but this time do not create a new container name but select the existing HDI container that has been created from the previous attempt  `MyHANAApp-dev`

1. Upon completion, the Database Connections will now show the service bound to the instance the wizard just created.

    ![Bound Connection](bound_connection.png)

1. We are now ready to deploy the development content into the database. Before you go ahead, we recommend that you increase the default number of scrollback lines in the integrated terminal, if you're using a Dev Space in SAP Business Application Studio. This is because there are many lines of log output about to be generated and you will want to see them all. So use menu path **File -> Preferences -> Settings** and search for the "Terminal › Integrated: Scrollback" setting. Set the value to 10000. Now, once you've done that, you're ready to deploy. Press the **Deploy** button (which looks like a rocket) at the **db** folder level in the SAP HANA Projects view.

    ![Deploy](deploy.png)

1. Scroll up to in the console to see what the build process has done.

> What is going on?
>
> CDS stands for Core Data Services. This is an infrastructure that allows you to create persistency and services using a declarative language. Notice how you are using the same syntax to define both the persistency and the services.
>&nbsp;
> You can find more information on CDS [in the help](https://help.sap.com/viewer/65de2977205c403bbc107264b8eccf4b/Cloud/en-US/855e00bd559742a3b8276fbed4af1008.html)
>
>You defined a CDS artifact, this is an abstraction, a design-time declaration of the entities to be represented in a database and the services to expose them.
>&nbsp;
> ![Build database module](cds1.png)
>
>The original `.cds` file was translated into `hdbtable`, which is the SQLDDL syntax specific to SAP HANA when you saved all of the files.
>&nbsp;
> ![Build database module](cds2.png)
>
>These `hdbtable` files were then translated into runtime objects such as tables in the HANA database.
>
> ![Build database module](cds3.png)
>
> If you checked the services in your space, you would see the service for your [HDI container](https://help.sap.com/viewer/4505d0bdaf4948449b7f7379d24d0f0d/latest/en-US/e28abca91a004683845805efc2bf967c.html).
>
> ![Build database module](console.png)
>
> You can find a similar example and further context on Core Data and Services in [this explanatory video](https://www.youtube.com/watch?v=UuXURLt1IQE&list=PL6RpkC85SLQAPHYG1x6IEu_exE5pa0UK_&index=7)

### Check the Database Explorer

You can now check the generated tables and views in the Database Explorer.

1. In the SAP HANA Projects view, press the **Open HDI Container** button

    ![Press Open HDI Container](open_hdi_container.png)

1. The Database Explorer will open in a new browser tab and automatically select the database entry for your project's HDI container.

1. Once open, navigate to the `Tables` section and click on the `Header` table.

    ![Open Table Definition](db_explorer1.png)

1. Note the name of the table matches the generated `hdbtable` artifacts. You will also see the physical schema managed by the HDI container.

    > Unless a name is specified during deployment, HDI containers are automatically created with names relative to the project and user generating them. This allows developers to work on different versions of the same HDI container at the same time.
    > ![Build database module](8.png)

### Load data into your tables

1. Download the [header file](https://raw.githubusercontent.com/SAP-samples/hana-opensap-cloud-2020/main/tutorial/app.interactions-Headers.csv) and the [items file](https://raw.githubusercontent.com/SAP-samples/hana-opensap-cloud-2020/main/tutorial/app.interactions-Items.csv) and the [texts file](https://raw.githubusercontent.com/SAP-samples/hana-opensap-cloud-2020/main/tutorial/app.interactions-Items.texts.csv) into your local file system.

1. Right-click again on the header table and choose **Import Data**.

    ![Import data](10.png)

1. Choose **Import Data** and press **Step 2**

    ![Import data](10_1.png)

1. Choose **Local** for the **Import Data From:** option. Browse for the `Header` file and click **Step 3**.

    ![Import data](11.png)

1. Keep the default import target and click **Step 4**.

    ![Import data](12.png)

1. Keep the default table mapping and press **Step 5**.

    ![Import data](13.png)

1. Keep the default error handling and press **Review**

    ![Import data](14.png)

1. Choose **Import into Database**.

    ![Import data](15.png)

1. You will see confirmation that 4 records have imported successfully.

    ![Import data](16.png)

1. Repeat the process with the `app.interactions-Items.csv` and `app.interactions-Items.texts.csv` files into the `ITEMS` and `ITEMS_TEXTS` tables.

    1![Import data](17.png)

### Check data loaded into the tables

1. You can now check the data loaded into the tables. Right-click on the `Items` table and click **Generate Select Statement**.

    ![Generate select statement](18.png)

1. Add the following WHERE clause to the SELECT statement and execute it to complete the validation below.

    ```SQL
    where "TEXT"  like '%happy%';
    ```
