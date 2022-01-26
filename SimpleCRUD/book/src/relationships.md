# Multi-table relationships

This section shows how to use SeaORM to perform operations between the Fruits table and a new suppliers table.

Creating relationships between tables can be verbose when doing so using code. Luckily, SeaORM makes this easy. Define a table called `suppliers` in the `fruit_markets` database by taking the following steps:

1. Login to mysql database using username and password created in the `installation` part of this tutorial in the previous section and switch to fruit_markets database.

   ```sql
   # Execute 
   use fruit_markets;
   ```

2. Create a table `suppliers` that references the primary key `fruit_id` of table `fruits`. This will show the type of fruit the supplier supplies to the fruit markets.

   ```sql
   CREATE TABLE suppliers(
       supplier_id INT NOT NULL AUTO_INCREMENT,
       supplier_name VARCHAR(255) NOT NULL,
       fruit_id  INT NOT NULL,  
       PRIMARY KEY (supplier_id),
       CONSTRAINT fk_fruits
       FOREIGN KEY (fruit_id) 
       REFERENCES fruits(fruit_id)
           ON UPDATE CASCADE
           ON DELETE CASCADE
   ) ENGINE=INNODB;
   ```

3. Use `sea-orm-cli` to generate the `Entity`, `Model`, `Relationship` and `ActiveModel`.

   ```sh
   $ sea-orm-cli generate entity -o src/suppliers_table -t suppliers
   ```

   A new directory `suppliers_table` is created in the `src` directory containing serveral files with code generated by `sea-orm-cli`.

4. Modify the `src/suppliers_table/prelude.rs` file to export memorable names of the `Entity, ActiveModel` etc

   ```rust
   - pub use super::suppliers::Entity as Suppliers;
   
   + pub use super::suppliers::{
   +     ActiveModel as SuppliersActiveModel, Column as SuppliersColumn, Entity as Suppliers,
   +     Model as SuppliersModel, PrimaryKey as SuppliersPrimaryKey, Relation as SuppliersRelation,
   + };
   
   ```

5. The `src/suppliers_table/suppliers.rs` contains errors indicating the `super::fruits` cannot be found in `supper`. This means the module is not exported properly. Fix this by importing the module:

   ```rust
   //! SeaORM Entity. Generated by sea-orm-codegen 0.5.0
   
   use sea_orm::entity::prelude::*;
   
   #[derive(Clone, Debug, PartialEq, DeriveEntityModel)]
   #[sea_orm(table_name = "suppliers")]
   pub struct Model {
       #[sea_orm(primary_key)]
       pub supplier_id: i32,
       pub supplier_name: String,
       pub fruit_id: i32,
   }
   
   #[derive(Copy, Clone, Debug, EnumIter, DeriveRelation)]
   pub enum Relation {
       #[sea_orm(
   -       belongs_to = "super::fruits::Entity",
   +		belongs_to = "crate::Fruits",
           from = "Column::FruitId",
   -       to = "super::fruits::Column::FruitId",
   +       to = "crate::FruitsColumn::FruitId",
           on_update = "Cascade",
           on_delete = "Cascade"
       )]
       Fruits,
   }
   
   - impl Related<super::fruits::Entity> for Entity {
   + impl Related<crate::Fruits> for Entity {
       ...
      }
   
   impl ActiveModelBehavior for ActiveModel {}
   ```

   `sea-orm-cli` automatically generates code to bind the `suppliers` table `Model` to the primary key of the `fruits` table using `belongs_to = "crate::Fruits",` `to = "crate::FruitsColumn::FruitId"` and `impl Related<crate::Fruits> for Entity`. This corresponds to the SQL query part

   ```sql
    CONSTRAINT fk_fruits
       FOREIGN KEY (fruit_id) 
       REFERENCES fruits(fruit_id)
           ON UPDATE CASCADE
           ON DELETE CASCADE
   ```

6. Import the module to the `src/main.rs` file

   ```rust
     mod fruits_table;
     use fruits_table::prelude::*;
   + mod suppliers_table;
   + use suppliers_table::prelude::*;
   
   // -- code snippet --
   
   // Convert this main function into async function
   #[async_std::main]
   async fn main() -> Result<()> {
       // -- code snippet --
   }
   ```

   

Next, `INSERT` and `SELECT` SQL queries on tables with foreign key using `SeaORM`.