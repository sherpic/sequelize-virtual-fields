# sequelize-virtual-fields.js

# Sequelize virtual fields magic

## What's it for?

This plugin for [Sequelize](http://sequelizejs.com/) adds some magic to VIRTUAL fields, so that they can be used the same as normal fields.

If a virtual field references attributes of an associated model, this can be defined in the model's definition and the required associations are loaded automatically by `Model#find()`.

Why is this useful? You might, for example, want to build a Drupal-style framework where every model instance has a 'name' field which which may take its value from fields in associated models.

## Current status

API is stable. Tested and works on MySQL. Untested on other dialects supported by Sequelize.

The results returned from `Model#find()` will change in v0.2 (see Roadmap below).

## Usage

Define the dependency of the virtual fields on other attributes or models:

	// define models
	var Person = sequelize.define('Person', { name: Sequelize.STRING });
	var Task = sequelize.define('Task', {
		name: Sequelize.STRING,
		nameWithPerson: {
			type: Sequelize.VIRTUAL,
			get: function() { return this.name + ' (' + this.Person.name + ')' }
			attributes: [ 'name' ],
			include: [ { model: Person, attributes: [ 'name' ] } ],
			order: [ ['name'], [ Person, 'name' ] ]
		}
	});
	
	// define associations
	Task.belongsTo(Person);
	Person.hasMany(Task);
	
	// activate virtual fields functionality
	sequelize.initVirtualFields();

Create some data:

	// create a person and task and associate them
	Promise.all({
		person: Person.create({ name: 'Brad Pitt' }),
		task: Task.create({ name: 'Do the washing' })
	}).then(function(r) {
		return r.task.setPerson(r.person);
	});

Then `find()` a task, referencing the virtual field:

	return Task.find({ attributes: [ 'nameWithPerson' ] })
	.then(function(task) {
		// task.values.nameWithPerson = 'Do the washing (Brad Pitt)'
	});

The associated model 'Person' has been automatically fetched in order to get the name of the person.

You can also sort by a virtual field:

	Task.findAll({ attributes: [ 'nameWithPerson' ], order: [ [ 'nameWithPerson' ] ] });

Please note that the behaviour above works because of the definition of `attribute`, `include` and `order` in the `Task` model's definition, as well as the getter function `get`.

## Tests

Use `npm test` to run the tests.
Requires a database called 'sequelize_test' and a db user 'sequelize_test', password 'sequelize_test'.

## Changelog

See changelog.md

## Roadmap

In next version, results from find will populate the values of virtual fields into `dataValues` and remove attributes/associated models that this plugin has added. This will make using virtual fields completely transparent to the user.

## Known issues

* Does not work with use of `association` in place of `model` in `include` or `order` e.g. `someModel.findAll({ include: [ {association: someAssociation } ] })` - throws error if encountered
* No support for `Sequelize.col()` in order clauses

If you discover a bug, please raise an issue on Github. https://github.com/overlookmotel/sequelize-virtual-fields/issues
