DBRefs
====================

DBRefs are properties of type `ObjectId` which refer to another documents `_id`
and can be `populate()`d when querying. An example is helpful:

    var mongoose = require('mongoose')
      , Schema = mongoose.Schema

    var PersonSchema = new Schema({
        name    : String
      , age     : Number
      , stories : [{ type: Schema.ObjectId, ref: 'Story' }]
    });

    var StorySchema = new Schema({
        _creator : { type: Schema.ObjectId, ref: 'Person' }
      , title    : String
      , fans     : [{ type: Schema.ObjectId, ref: 'Person' }]
    });

    var Story  = mongoose.model('Story', StorySchema);
    var Person = mongoose.model('Person', PersonSchema);

So far we've created two models. Our `Person` model has it's `stories` field
set to an array of `DBRefs`. The `ref` option is what tells Mongoose which
model this DBRef refers to, in our case the `Story` model. All `_id`s we
store here must be document `_id`s from the `Story` model. We also added
a `_creator` DBRef to our `Story` schema which refers to a single `Person`.

## Querying DBRefs

    var aaron = new Person({ name: 'Aaron', age: 100 });

    aaron.save(function (err) {
      if (err) ...

      var story1 = new Story({
          title: "A man who cooked Nintendo"
        , _creator: aaron._id
      });

      story1.save(function (err) {
        if (err) ...
      });
    })

So far we haven't done anything special. We've merely created a `Person` and
a `Story`. Now let's take a look at populating our story's `_creator`:

    Story
    .findOne({ title: /Nintendo/i })
    .populate('_creator') // <--
    .run(function (err, story) {
      if (err) ..
      console.log('The creator is %s', story._creator.name);
      // prints "The creator is Aaron"
    })

Yup that's it. We've just queried for a `Story` with the term Nintendo in it's
title and also queried the `Person` collection for the story's creator. Nice!

Arrays of DBRefs work the same way. Just call the `populate` method on the query and
an array of documents will be returned in place of the `ObjectId`s.

What if we only want a few specific fields returned for the DBRef query? This can
be accomplished by passing an array of field names to the `populate` method:

    Story
    .findOne({ title: /Nintendo/i })
    .populate('_creator', ['name']) // <-- only return the Persons name
    .run(function (err, story) {
      if (err) ..

      console.log('The creator is %s', story._creator.name);
      // prints "The creator is Aaron"

      console.log('The creators age is %s', story._creator.age)
      // prints "The creators age is null'
    })

Now this is much better. The only property of the creator we are using
is the `name` so we only returned that field from the db. Efficiency FTW!

## Updating DBRefs

Now that we have a story we realized that the `_creator` was incorrect. We can
update `DBRef`s the same as any other property through the magic of Mongooses
internal casting:

    var guille = new Person({ name: 'Guillermo' });
    guille.save(function (err) {
      if (err) ..

      story._creator = guille; // or guille._id

      story.save(function (err) {
        if (err) ..

        Story
        .findOne({ title: /Nintendo/i })
        .populate('_creator', ['name'])
        .run(function (err, story) {
          if (err) ..

          console.log('The creator is %s', story._creator.name)
          // prints "The creator is Guillermo"
        })

      })
    })