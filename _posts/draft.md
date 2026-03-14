

# SwiftRide – Building a Ride Sharing App (Part 2 – Creating the Models)

In the [last part](/_posts/2026-03-11-swiftride-setting-up-the-stack.md) we setup the stack for our application. This included creating the database (without tables) and installing most of the required node packages that we will be using in our application. In this part, we will move things one step further and create our tables and the associated models. 

### Sequelize Migrations  

You may be very tempted to open your newly created database with a visual tool like Base or BeeKeeper etc and add tables, manually. Please don't. Since we are using Sequelize ORM, we need to use Sequelize migrations to configure the schema of the database. If you start changing the tables using a visual designer then Sequelize will loose all tracking and it will not know what is going on.  

There are various ways of creating your migration file. You can create a new migration empty template using the Sequelize terminal command. 

``` javascript 
npx sequelize-cli migration:generate --name create-user
```

This will only create the migration file. But in this particular case, I also want to create the associated `User` model. So, we will use a different Sequelize command. 

``` javascript 
npx sequelize-cli model:generate --name User --attributes username:string,password:string,role:string
```

This will not only create the `User` model with `username`, `password` and `role` attributes but also a migration file to create the table associated with the `User` model. 

> In Sequelize tables are named after the name of the model but they are pluralized. This means if your model name is User then in the database Sequelize will create a table called Users. 

The migration file is shown below: 

``` swift 
'use strict';
/** @type {import('sequelize-cli').Migration} */
module.exports = {
  async up(queryInterface, Sequelize) {
    await queryInterface.createTable('Users', {
      id: {
        allowNull: false,
        autoIncrement: true,
        primaryKey: true,
        type: Sequelize.INTEGER
      },
      username: {
        type: Sequelize.STRING
      },
      password: {
        type: Sequelize.STRING
      },
      role: {
        type: Sequelize.STRING
      },
      createdAt: {
        allowNull: false,
        type: Sequelize.DATE
      },
      updatedAt: {
        allowNull: false,
        type: Sequelize.DATE
      }
    });
  },
  async down(queryInterface, Sequelize) {
    await queryInterface.dropTable('Users');
  }
};
```

Keep in mind that this migration file, along with all the code in the file is created automatically by running the following command from the terminal. 

``` javascript 
npx sequelize-cli model:generate --name User --attributes username:string,password:string,role:string
```

If you run this migration then it will create the `Users` table in the `swiftridedb` database. You can use any Postgres visual tool to look at the database. I recommend [BeeKeeper](https://www.beekeeperstudio.io/get). BeeKeeper is free for MacOS, Linux and Windows.  

![BeeKeeper](../images/bee-keeper.png)

Each migration has two different methods. The up method is executed when you run the migration and the down method is executed when you undo a migration. Whatever you do in the up method, just remember to do the opposite in the down method. 

> I have on purpose made the role attribute as a string instead of a roldId. In the next section we will add a `Roles` table and also update our `Users` table schema through migration to support roleId, instead of the role name. 

### Creating Roles Table and Adding Relationship Between Users and Roles 

Our next step is to add `Roles` table and also update the `Users` table to have a relationship with the `Roles` table through roleId. This can be done in multiple steps. First we can use the Sequelize command to create the Role model and the associated migration file. This is shown below: 

``` javascript 
npx sequelize-cli model:generate --name Role --attributes name:string
```

Next, we can write a custom migration which will remove the role column from `Users` table and add a new `roleId` column. The `roleId` column will serve as a foreign key in `Users` table and will be linked to the `id` column in the `Roles` table. 

The migration is implemented below: 

``` swift 
'use strict';

module.exports = {
  async up(queryInterface, Sequelize) {

    // 1️⃣ Add roleId column
    await queryInterface.addColumn('Users', 'roleId', {
      type: Sequelize.INTEGER,
      allowNull: false,
      references: {
        model: 'Roles',
        key: 'id'
      },
      onUpdate: 'CASCADE',
      onDelete: 'RESTRICT'
    });

    // 2️⃣ Remove old role column
    await queryInterface.removeColumn('Users', 'role');
  },

  async down(queryInterface, Sequelize) {

    // 1️⃣ Add back role column
    await queryInterface.addColumn('Users', 'role', {
      type: Sequelize.STRING,
      allowNull: false
    });

    // 2️⃣ Remove roleId column
    await queryInterface.removeColumn('Users', 'roleId');
  }
};
```

Let's discuss some of the key pieces of this migration. 

The `up` function is executed when the migration is applied. In this case, it first adds a new `roleId` column to the `Users` table. This column is of type `INTEGER` because it will store the primary key value from the `Roles` table. The `references` property tells Sequelize that `roleId` is a foreign key connected to the `id` column in the `Roles` table. This means every value stored in `Users.roleId` must correspond to a valid role in the `Roles` table.

The `allowNull: false` rule ensures that every user must have a role. A user cannot exist without being linked to one of the records in the `Roles` table. The `onUpdate: 'CASCADE'` option means that if the `id` of a role ever changes in the `Roles` table, the related `roleId` values in the `Users` table will be updated automatically. The `onDelete: 'RESTRICT'` option prevents deleting a role if it is still being used by one or more users. This protects the integrity of your data and avoids orphaned records.

After adding the `roleId` column, the migration removes the old `role` column from the `Users` table. Previously, the role was stored as plain text such as `"rider"` or `"driver"`. While that works for very small projects, it is not a great long term design. Storing role values as strings can lead to inconsistencies like `"Driver"`, `"driver"`, or `"drivers"`. By moving roles into their own table, the database becomes more structured and easier to maintain.

The `down` function does the opposite. It is executed when the migration is rolled back. It recreates the old `role` column as a string and then removes the `roleId` column. This allows you to undo the schema change if needed.

One important thing to understand is that this migration only changes the table structure. It does not automatically move existing role values from the old `role` column into the new `roleId` column. If your `Users` table already contains data, then you would usually need an additional step to populate the `Roles` table first and then update each user so that their `roleId` points to the correct role record before removing the old `role` column.

This migration is a step toward a more normalized database design. Instead of storing repeated role strings directly in the `Users` table, each user now points to a single record in the `Roles` table. This makes the design more scalable, easier to query, and much safer as your SwiftRide application grows.

At present none of the database tables have been created. We have implemented our migrations but we still have not executed any of them. In the next section, we will discuss how to run our migrations. 

### Running Migrations and Creating Model Associations 

Migrations can be executed by using Sequelize terminal commands. Since, we have not run a single migration we can run `sequelize db:migrate`. This command will run all the migrations that have not been executed before. 

Once the migrations have been executed, you can check your database and see all the new tables being created. This includes `Users` and `Roles` table. You will also notice a table called `SequelizeMeta` table. That table is automatically created by Sequelize and its main purpose is to track, which migrations have already been executed. Do not add or delete anything from this table or Sequelize will become confuse as which, migrations have already been executed. 

Although the appropriate tables have been created but the Sequelize model files still needs to be updated... manually. Open the `User.js` file located in the models folder of your project. 

Below you can see the `User.js` file with all the associations and the newly added `roleId`. 

``` javascript 
'use strict';
const {
  Model
} = require('sequelize');
module.exports = (sequelize, DataTypes) => {
  class User extends Model {
    /**
     * Helper method for defining associations.
     * This method is not a part of Sequelize lifecycle.
     * The `models/index` file will call this method automatically.
     */
    static associate(models) {

      models.User.belongsTo(models.Role, {
        foreignKey: 'roleId', 
        as: 'role'
      })

    }
  }
  User.init({
    username: DataTypes.STRING,
    password: DataTypes.STRING,
    roleId: DataTypes.INTEGER 
  }, {
    sequelize,
    modelName: 'User',
  });
  return User;
};
```

The associate function is where relationships between models are defined. Sequelize automatically calls this method when the application starts, allowing you to declare how tables are connected.

The line:

``` javascript 
models.User.belongsTo(models.Role, {
  foreignKey: 'roleId',
  as: 'role'
});
```

defines a relationship where each User belongs to a single Role. The roleId column in the Users table acts as a foreign key that references the id column in the Roles table. This establishes the connection between the two tables.

The foreignKey option explicitly tells Sequelize which column is used for the relationship. The as: 'role' alias provides a convenient name that can be used when loading related data.

For example, if you want to retrieve users along with their roles, you can write a query like this:

``` javascript 
User.findAll({
  include: {
    model: Role,
    as: 'role'
  }
});
```

Sequelize will automatically perform the necessary SQL join and return the role information along with each user.

By defining these associations at the model level, Sequelize understands how your tables relate to each other. This allows you to work with relationships using JavaScript instead of manually writing SQL joins. As the application grows, this approach makes the backend easier to maintain and keeps your database structure organized.