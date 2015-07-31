# RWS Meteor App Issues

Outline detailing the structuring of the RWS Meteor app, current issues, recommended solutions and code examples.

## Core Pieces

* User Authentication / Creation
  * Roles
    * Teacher
    * Student
* Shopify API
* Forms





## User Authentication / Creation

The app has been refactored to extend the `User` model with `collection2` and the `teacher` and `student` collections have been removed. 
Instead they are simply roles of the default user model. Currently the process of creeating a user is messy, and two many actions are being nested into one function. 
They should be fragmented for better understanding and troubleshooting when a piece does not perform as desired.

The flow for a new user (teacher) creation will be:

1. Signup form submit
2. Form validation
3. Create meteor user
  * Create shopify custom collection, returns new collection id
  * Update user with collection ID
  * Create base/template documents for user settings (account/store/design/foundation)
4. Send the user an email
5. Login the user 

[Suggested solution via Stackoverflow](http://stackoverflow.com/questions/30050159/accounts-oncreateuser-adding-extra-attributes-while-creating-new-users-good-pra) - leverages `onCreateUser`

##### Psuedo Code


```js
// Form submit
Template.formSignup.events({
  'submit form': function (event) {

    // Create Shopify Custom Collection
    Meteor.call('createShopifyCustomCollection', function (err, res) {
      if ( err ) {
        // Validate
      } else {
        var collection_id = res.collection_id
        
        // Creat Account
        Accounts.createUser({
          email : event.target.email.value,
          password : event.target.password.value,
          collection_id : collection_id
        })

        // Accounts.onCreateUser Hook runs
      }
    })
  }
})

// onCreateUser hook
Accounts.onCreateUser(function (options,user) {
  if ( options.profile ) { ... }
  
  // Teacher
  if ( options.role === 'teacher' ) {
    // Customize for teacher
    createUserCollection()
  }

  // Student
  if ( options.role === 'student' ) {
    // Customize for student
  }

  return user
})

// Create User Collections
createUserCollection = function() {
  Stores.insert({ ... })
  Settings.insert({ ... })
}
```

Note that the `accounts-password` package makes a number of methods available:

* `Accounts.createUser()`
* `Accounts.changePassword()`
* `Accounts.forgotPassword()`
* `Accounts.resetPassword()`
* `Accounts.setPassword()`
* `Accounts.verifyEmail()`


###### Other Notes

- It is possible to piggyback functions using underscores wrap. [See this StackOverflow solution](http://stackoverflow.com/questions/12984637/is-there-a-post-createuser-hook-in-meteor-when-using-accounts-ui-package)



## Form Validation

Currently a large issue is the lack of/trouble with form validation. Meteor's standard errors are not helpful in performing validation in the DOM/client. 
`collection2` provides an `invalidKeys` object in the error response if the Schema is set to use it.
There are several libraries we can use to help with this, however it's important to understand the two-sided approach of validation.

#### Client

- Make sure email is in fact an email with regex, etc (contains an @, a .com or .org, etc)
- No empty fields
- Required fields

#### Server

- Email already exists in DB
- Store name already taken

[Suggested solution via StackOverflow](http://stackoverflow.com/a/26239457/1132995)

Which uses a combination of

* [meteor-simple-schema](https://github.com/aldeed/meteor-simple-schema)
* [meteor-collection2](https://github.com/aldeed/meteor-collection2)
* [meteor-autoform](https://github.com/aldeed/meteor-autoform)

Another alternative would be [Mesophere](https://atmospherejs.com/copleykj/mesosphere) - A dual client/server side form data validation, transformation, and aggregation package for Meteor