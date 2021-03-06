# [Red's Inventory Management](https://arpadgbondor.github.io/Red-Inventory-Management/)
- [Program features](#program-features)
  - [Setup database connection](#setup-database-connection)
  - [Login](#login) 
  - [Notifications](#notifications)
  - [Program log](#program-log)
  - [Tables](#tables)
  - [Lists](#lists)
- [Program structure](#program-structure)
  - [Entity Layer](#entity-layer)
  - [Data Layer](#data-layer)
  - [Business Layer](#business-layer)
  - [UI Layer](#ui-layer)
    - [Setup database connection UI](#setup-database-connection-ui)
    - [Login UI](#login-ui)
    - [Main window UI](#main-window-ui)
    - [Tables UI](#tables-ui)
    - [Lists UI](#lists-ui)

## Program features

### Setup database connection
The program connects to a database file with ".mdf" extension using MS SQL LocalDB. [Microsoft SQL Server 2014 Express LocalDB](https://github.com/kjbartel/SqlLocalDB2014-Bootstrapper) is ~~set as prerequisite in the Setup project~~, but the program works with other versions of LocalDb as well. (Update: Setup project has been removed, because it's not supported in new versions of Visual Studio.)
When the application starts first time, a small window pops up and you have to create a new database file, or connect to an existing one. You can't use the program without database connection. The program automatically stores the path of the database folder and the name of the database file in the DatabaseSettings.txt file in the application folder. You can change the database connection settings under the Settings => Database menu.

### Login
The program stores usernames and passwords in the "Users" table, and the passwords are encrypted with salted SHA-256 hashing. If the "Users" table is empty at the start of the program, a small window pops up, and you have to add the first user to the database. If there is at least one user in the "Users" table, you can enter the username and the password in the login window. You can Add/Modify/Remove users under the Settings => Users menu.

### Notifications
The program sends messages to the user via Notifications in the top right corner of the screen.
The notifications are based on this [GitHub project](https://github.com/IvanLeonenko/WPFGrowlNotification)

### Program log
The program uses log4net to log it's behaviour into the RedLog.txt file. To see more detailed debug information, open the programs config file, and  <-- comment --> the filter section from the log4net appender tags.

### Tables
Every table builds up in a similar way in the program with the same features.
You can Add/Edit/Delete:
- Products
- Partners
- Incoming transactions
- Outgoing transactions
- Users

If you Add/Edit a record a smaller window pops up in the center of the screen. You can change the details of the record, and you can save or cancel the changes with the two button at the bottom of the window.

### Lists

#### Inventory list
- This list shows every product's stock quantities. If you select a product, and press the "Quantity details" button above the list, a new window pops up, that lists every transaction, that affected the product's stock quantity.

#### Partner transactions
- This list shows every partner's total money transactions. If you select a partner, and press the "partner transactions" button above the list, a new window pops up, that lists every money transaction with that partner.

## Program structure

### Entity Layer
The Entity Layer contains the classes responsible for the structure of the database tables, and the structure of the lists passed between the layers.

#### Datatable Entities
- PartnerEntity class
- ProductCategoryEntity class
- ProductEntity class
- TransactionBodyEntity class
- TransactionHeadEntity class
- UserEntity class

#### List Entities
- ProductListEntity class
- TransactionBodyListEntity class
- TransactionHeadListEntity class

### Data Layer
The Data Layer contains the classes responsible for the Database connection, and the data provider classes responsible for the consistent database state.

#### Database connection
- InventoryContext class
  - Database context for the Inventory database.
  - Derived from the Entity Framework's System.Data.Entity.DbContext class.
- DatabaseConnection class
  - Generates and holds the connectionstring to the mdf file.
  - Uses Entity Framework to connect to the MS SQL LocalDB.
  - Can create a database file if the file does not exists.
  - Automatically migrates database to the latest version after connecting to it.
  - Contains template "linq to sql" database manipulation functions, that can be used in the data provider classes

#### Data providers
- PartnerProvider class
- ProductCategoryProvider class
- ProductProvider class
- TransactionProvider class
- UsersProvider class

### Business Layer
The classes in the Business Layer are providing services used by the UI Layer and connecting the Data Layer and the UI Layer.
- DatabaseConnection class
  - Reads the default setting from the "DatabaseSettings.txt" and tries to connect to the default file when TestConnection() is called.
  - Connects the DatabaseConnection class in the Data Layer with the UI Layer.
- UserLogin class
  - Handles user authentication, and holds the last logged in user's ID.
  - Connects the UsersProvider class in the Data Layer with the UI Layer
- ManagePartners class
  - Connects the PartnerProvider class in the Data Layer with the UI Layer
- ManageProducts class
  - Connects the ProductCategoryProvider and ProductProvider classes in the Data Layer with the UI Layer
- ManageTransactions class
  - Connects the TransactionProvider class in the Data Layer with the UI Layer

### UI Layer
The UI Layer was made by using MVVM pattern.

#### Setup database connection UI
- ViewModel: SetupConnectionViewModel
- View:
  - SetupConnectionWindow (Called from MainWindowViewModel constructor)
  - SetupConnectionMenuView (Called from Settings => Database)
  - SetupConnectionView

#### Login UI
  - New User
    - ViewModel: NewUserViewModel
    - View: NewUserWindow (Called from MainWindowViewModel constructor and Settings => Users table)
  - Edit User
    - ViewModel: EditUserViewModel
    - View: EditUserWindow (Called from Settings => Users table)
  - Login
    - ViewModel: LoginViewModel
    - View: LoginWindow (Called from MainWindowViewModel constructor)

#### Main window UI
- ViewModel: MainWindowViewModel
- View: MainWindow

#### Tables UI
- View: TableView
  - Every table uses the same view, which contains the basic table features: Title, New-, Edit- and Delete-button
  - The middle of the view is a ContentControl, that shows different views based on the different ViewModels
- Model: TableModel\<Entity> (derived from ListModel\<Entity>)
  - This is the base class of every TableViewModel class, and it implements everything that connects the viewmodel with the TableView, except 4 abstract function, that has to be implemented in the derived classes:
    - NewItem()
    - EditItem()
    - DeleteItem()
    - RefreshList()
- Partners table
  - ViewModel: PartnersViewModel (derived from TableModel\<PartnerEntity>)
  - View: PartnersTableView (in the ContentControl of TableView)
- Product categories table
  - ViewModel: ProductCategoriesViewModel (derived from TableModel\<ProductCategoryEntity>)
  - View: ProductCategoriesTableView (in the ContentControl of TableView)
- Products table
  - ViewModel: ProductsViewModel (derived from TableModel\<ProductListEntity>)
  - View: ProductsTableView (in the ContentControl of TableView)
- Transactions table
  - ViewModel: TransactionsViewModel (derived from TableModel\<TransactionHeadListEntity>)
  - View: TransactionsTableView (in the ContentControl of TableView)
- Users table
  - ViewModel: UsersViewModel (derived from TableModel\<UserEntity>)
  - View: UsersTableView  (in the ContentControl of TableView)


##### Add/Edit table-records
- View: EditItemWindow
  - Every table uses the same edit view. A Window pops up in the middle of the screen with a(n) ADD/Save button and a Cancel Button at the bottom of the window.
  - The middle of the view is a ContentControl, that shows different views based on the different ViewModels
- Model: EditItemModel\<Entity>
  - This is the base class of every Edit*ViewModel class, and it implements everything that connects the viewmodel with the EditItemWindow, except one logical abstract function, that has to be implemented in the derived classes:
    - Save()
- Add/Edit partner
  - ViewModel: EditPartnerViewModel (derived from EditItemModel\<PartnerEntity>)
  - View: EditPartnerView (in the ContentControl of EditItemWindow)
- Add/Edit product category
  - ViewModel: EditProductCategoryViewModel (derived from EditItemModel\<ProductCategoryEntity>)
  - View: EditProductCategoryView (in the ContentControl of EditItemWindow)
- Add/Edit product
  - ViewModel: EditProductViewModel (derived from EditItemModel\<ProductListEntity>)
  - View: EditProductView (in the ContentControl of EditItemWindow)
- Add/Edit transaction
  - ViewModel: EditTransactionViewModel (derived from EditItemModel\<TransactionHeadListEntity>)
  - View: EditTransactionView (in the ContentControl of EditItemWindow)
- Add/Edit user
  - Adding and editing users are handled by the [Login UI](#login-ui)

#### Lists UI
- View: ListView
  - Every list uses the same view.
  - The middle of the view is a ContentControl, that shows different views based on the different ViewModels
- Model: ListModel\<Entity>
  - This is the base class of every ListViewModel class and DetailsViewModel class, and it implements everything that connects the viewmodel with the ListView, except one abstract function, that has to be implemented in the derived classes:
    - RefreshList()
- Inventory list
  - ViewModel: InventoryViewModel (derived from ListModel\<TransactionBodyListEntity>)
  - View: InventoryListView (in the ContentControl of ListView)
- Partner transactions list
  - ViewModel: PartnerTransactionsViewModel (derived from ListModel\<TransactionHeadListEntity>)
  - View: PartnerTransactionsListView (in the ContentControl of ListView)

##### Details list
- View: ListDetailsWindow
  - Every details list uses the same view.
  - The middle of the view is a ContentControl, that shows different views based on the different ViewModels
- Inventory details list
  - ViewModel: InventoryDetailsViewModel (derived from ListModel\<TransactionHeadListEntity>)
  - View: InventoryListDetailsView (in the ContentControl of ListDetailsWindow)
- Partner transaction details list
  - ViewModel: PartnerTransactionsDetailsViewModel (derived from ListModel\<TransactionHeadListEntity>)
  - View: PartnerTransactionsListDetailsView (in the ContentControl of ListDetailsWindow)
