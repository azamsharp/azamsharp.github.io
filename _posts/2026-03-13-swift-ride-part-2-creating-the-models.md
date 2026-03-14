

# SwiftRide – Building a Ride Sharing App (Part 2 – Creating the Models)

![Swift Ride Part 2](/images/swiftride-part-2.png)

In the [last part](https://azamsharp.com/2026/03/11/swiftride-setting-up-the-stack.html) we focused on setting up the foundation for the application. We created the PostgreSQL database (without any tables yet) and installed the core Node packages that our backend will rely on. Think of that step as preparing the workspace before we start building anything meaningful.

In this part, we are going to move the project forward by defining the actual structure of our data. We will create the database tables using Sequelize migrations and generate the corresponding models that our application will use to interact with those tables. This is where the backend starts to take shape, because once the models and tables are in place, the application finally has a proper data structure to work with.

<!-- Book Banner: SwiftUI Architecture Book -->
<div class="azam-book-banner" role="region" aria-label="SwiftUI Architecture Book Banner">
  <div class="azam-book-banner__inner">
    <div class="azam-book-banner__cover">
      <!-- Replace the src with your real book cover image -->
      <img
        src="https://azamsharp.school/images/swiftui-architecture-book-cover.png"
        alt="SwiftUI Architecture book cover"
        loading="lazy"
      />
    </div>
    <div class="azam-book-banner__content">
      <p class="azam-book-banner__eyebrow">SwiftUI Architecture Book</p>
      <h3 class="azam-book-banner__title">Patterns and Practices for Building Scalable Applications</h3>
      <p class="azam-book-banner__subtitle">
        A practical guide to building SwiftUI apps that stay clean as they grow.
      </p>
      <div class="azam-book-banner__actions">
        <a class="azam-book-banner__button" href="https://azamsharp.school/swiftui-architecture-book.html" target="_blank" rel="noopener">
          Get the book
        </a>
      </div>
    </div>
  </div>
</div>

<style>
  .azam-book-banner {
    --bg1: #0b1220;
    --bg2: #111a2d;
    --text: rgba(255, 255, 255, 0.92);
    --muted: rgba(255, 255, 255, 0.74);
    --border: rgba(255, 255, 255, 0.12);
    --shadow: 0 18px 45px rgba(0, 0, 0, 0.28);
    --accent: #6ee7b7; /* tweak to match your brand */
    --accent2: #60a5fa;

    margin: 22px 0;
    color: var(--text);
    border: 1px solid var(--border);
    border-radius: 16px;
    overflow: hidden;
    background: radial-gradient(1200px 600px at 10% 0%, rgba(96, 165, 250, 0.22), transparent 60%),
                radial-gradient(900px 500px at 90% 30%, rgba(110, 231, 183, 0.18), transparent 60%),
                linear-gradient(135deg, var(--bg1), var(--bg2));
    box-shadow: var(--shadow);
  }

  .azam-book-banner__inner {
    display: grid;
    grid-template-columns: 132px 1fr;
    gap: 18px;
    padding: 18px;
    align-items: center;
  }

  .azam-book-banner__cover {
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .azam-book-banner__cover img {
    width: 132px;
    height: auto;
    border-radius: 12px;
    border: 1px solid rgba(255,255,255,0.14);
    box-shadow: 0 14px 28px rgba(0,0,0,0.35);
    background: rgba(255,255,255,0.04);
  }

  .azam-book-banner__eyebrow {
    margin: 0 0 6px 0;
    font-size: 12px;
    letter-spacing: 0.08em;
    text-transform: uppercase;
    color: var(--muted);
  }

  .azam-book-banner__title {
    margin: 0 0 8px 0;
    font-size: 18px;
    line-height: 1.25;
  }

  .azam-book-banner__subtitle {
    margin: 0 0 14px 0;
    font-size: 14px;
    line-height: 1.55;
    color: var(--muted);
    max-width: 62ch;
  }

  .azam-book-banner__actions {
    display: flex;
    gap: 12px;
    align-items: center;
    flex-wrap: wrap;
  }

  .azam-book-banner__button {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    padding: 10px 14px;
    border-radius: 12px;
    font-weight: 700;
    font-size: 14px;
    color: #071018;
    text-decoration: none;
    background: linear-gradient(135deg, var(--accent), var(--accent2));
    border: 0;
    box-shadow: 0 10px 22px rgba(0,0,0,0.28);
    transition: transform 140ms ease, filter 140ms ease;
  }

  .azam-book-banner__button:hover {
    transform: translateY(-1px);
    filter: brightness(1.02);
  }

  .azam-book-banner__link {
    font-size: 14px;
    color: rgba(255,255,255,0.86);
    text-decoration: none;
    border-bottom: 1px solid rgba(255,255,255,0.22);
    padding-bottom: 2px;
    transition: border-color 140ms ease, color 140ms ease;
  }

  .azam-book-banner__link:hover {
    color: rgba(255,255,255,0.95);
    border-color: rgba(255,255,255,0.45);
  }

  /* Mobile */
  @media (max-width: 520px) {
    .azam-book-banner__inner {
      grid-template-columns: 1fr;
      text-align: left;
    }

    .azam-book-banner__cover {
      justify-content: flex-start;
    }

    .azam-book-banner__cover img {
      width: 120px;
    }
  }
</style>

### Sequelize Migrations  

At this point you might feel tempted to open your database using a visual tool like Base, TablePlus, or BeeKeeper and start creating tables manually. Resist that temptation. Since we are using Sequelize as our ORM, the database schema should always be managed through Sequelize migrations. If you start modifying tables directly using a visual designer, Sequelize will lose track of the schema changes and things can quickly become messy.

Migrations allow you to define the structure of your database in code. They also give you a history of how the database evolved over time, which is extremely useful when working with multiple developers or deploying your application to different environments.

There are a couple of ways to create a migration file. The simplest option is to generate an empty migration template using the Sequelize CLI:

``` javascript 
npx sequelize-cli migration:generate --name create-user
```

This command creates a migration file, but it does not create the associated Sequelize model. In this case we want both the model and the migration file, so we will use a different command:

``` javascript 
npx sequelize-cli model:generate --name User --attributes username:string,password:string,role:string
```

This command does two things. First, it creates the User model with the attributes username, password, and role. Second, it generates a migration file that will create the corresponding table in the database.

One thing to keep in mind is how Sequelize names tables. By default, Sequelize pluralizes the model name. So if your model is called User, the actual table created in the database will be named Users.

The generated migration file looks like this:

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

It is important to understand that this entire file is automatically generated by running the following command:

``` javascript 
npx sequelize-cli model:generate --name User --attributes username:string,password:string,role:string
```

When this migration is executed, it will create the Users table inside the swiftridedb database.

If you want to inspect the database visually, you can use any PostgreSQL client. My personal recommendation is [BeeKeeper](https://www.beekeeperstudio.io/get). It is free and available on macOS, Linux, and Windows. 

![BeeKeeper](../images/bee-keeper.png)

Every migration contains two methods: up and down. The up method runs when the migration is applied and is responsible for creating or modifying database structures. The down method does the opposite and is executed when you roll back the migration. As a general rule, whatever you do in the up method should be reversed in the down method.

You may have noticed that the role attribute in the Users table is currently stored as a string. I did this intentionally. In the next section we will introduce a proper Roles table and then update the Users table through another migration so that it stores a roleId instead of the role name. This will give us a cleaner and more scalable database design.

### Creating Roles Table and Adding Relationship Between Users and Roles 

Our next step is to introduce a Roles table and update the Users table so that it references roles through a roleId. Instead of storing the role name directly in the Users table, each user will point to a record in the Roles table. This gives us a cleaner and more flexible database design.

We will start by generating the Role model and its associated migration using the Sequelize CLI:

``` javascript 
npx sequelize-cli model:generate --name Role --attributes name:string
```

This command creates the Role model and a migration file that will generate the Roles table in the database.

Next, we will write a custom migration to modify the Users table. The goal is to remove the existing role column and replace it with a roleId column. This new column will act as a foreign key and will reference the id column in the Roles table.

The migration looks like this:

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

At this point, we have written all of our migrations, but nothing has actually been created in the database yet. To execute the migrations, run the following command from the terminal:

``` javascript 
npx sequelize-cli db:migrate
```

Since this is the first time we are running migrations for the project, Sequelize will execute every migration that has not been applied yet. In our case, that means it will create the Users table, the Roles table, and apply the custom migration that updates Users to use roleId.

Once the migrations finish running, open your database using a tool like BeeKeeper and inspect the schema. You should now see the tables we created, including Users and Roles. You will also notice another table called SequelizeMeta. This table is automatically managed by Sequelize and is used to keep track of which migrations have already been executed.

That table may not look very exciting, but it is extremely important. Sequelize checks SequelizeMeta before running migrations so it knows which ones are new and which ones have already been applied. In other words, this is how Sequelize keeps itself organized. Do not manually insert, delete, or edit rows in this table. If you do, Sequelize can lose track of the migration history and your database state can get out of sync very quickly.

Even though the database tables now exist, there is still one more step left. The Sequelize model files need to reflect the same relationships we created in the database. This part is done manually.

Open the User.js file inside the models folder and update it so it includes the new roleId field and the relationship to the Role model.

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

The associate function is where model relationships are defined. Sequelize will call this function when your application starts, and this is where you tell Sequelize how one model is connected to another.

This line is doing the heavy lifting:

``` javascript 
models.User.belongsTo(models.Role, {
  foreignKey: 'roleId',
  as: 'role'
});
```

It tells Sequelize that each User belongs to a single Role. The relationship is based on the roleId column in the Users table, which points to the id column in the Roles table.

The foreignKey option makes it explicit which column is being used for the relationship. The as: 'role' part gives the association a friendly name, which becomes very useful when loading related data.

For example, if you want to fetch all users along with their role information, you can write:

``` javascript 
User.findAll({
  include: {
    model: Role,
    as: 'role'
  }
});
```

Sequelize will take care of generating the join behind the scenes and return each user with its associated role attached.

This is one of the biggest benefits of setting up associations correctly. Instead of constantly thinking about joins and foreign keys when writing queries, you work at the model level and let Sequelize handle the SQL for you.

So at this stage, we have done three important things. We created the tables through migrations, updated the schema to support proper relationships, and taught Sequelize how those models connect to each other. That gives us a much stronger foundation than throwing random columns into a database and hoping for the best.

In the next part, we can finally start using these models to build actual features for SwiftRide.

### Conclusion

At this point our database foundation is in place. We created the `Users` and `Roles` tables using Sequelize migrations, established a proper foreign key relationship between them, and configured the model associations so Sequelize understands how these tables relate to each other.

Using migrations instead of manually editing the database is extremely important in a real project. Migrations give you a reproducible history of how the database evolved. If another developer joins the project or you deploy the application to a new environment, running the migrations will recreate the exact same schema automatically. This keeps the database structure consistent across development, staging, and production.

We also improved the database design by moving roles into their own table. Instead of storing role names as plain text in the `Users` table, each user now references a role through `roleId`. This type of relational design makes the database easier to maintain and prevents inconsistent data.

In the next part of the series, we will start building the authentication layer for SwiftRide. We will implement user registration, password hashing, and login functionality, allowing users to create accounts and securely access the application.
