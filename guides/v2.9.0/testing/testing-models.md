_Unit testing methods and computed properties follows previous patterns shown
in [Unit Testing Basics] because DS.Model extends Ember.Object._

[Ember Data] Models can be tested using the `moduleForModel` helper.

Let's assume we have a `Player` model that has `level` and `levelName`
attributes. We want to call `levelUp()` to increment the `level` and assign a
new `levelName` when the player reaches level 5.

> You can follow along by generating your own model with `ember generate
> model player`.

```javascript {data-filename=app/models/player.js}
import DS from 'ember-data';

export default DS.Model.extend({
  level:     DS.attr('number', { defaultValue: 0 }),
  levelName: DS.attr('string', { defaultValue: 'Noob' }),

  levelUp() {
    var newLevel = this.incrementProperty('level');
    if (newLevel === 5) {
      this.set('levelName', 'Professional');
    }
  }
});
```

Now let's create a test which will call `levelUp` on the player when they are
level 4 to assert that the `levelName` changes. We will use `moduleForModel`:

```javascript {data-filename=tests/unit/models/player-test.js}
import { moduleForModel, test } from 'ember-qunit';
import Ember from 'ember';

moduleForModel('player', 'Unit | Model | player', {
  // Specify the other units that are required for this test.
  needs: []
});

test('should increment level when told to', function(assert) {
  // this.subject aliases the createRecord method on the model
  const player = this.subject({ level: 4 });

  // wrap asynchronous call in run loop
  Ember.run(() => player.levelUp());

  assert.equal(player.get('level'), 5, 'level gets incremented');
  assert.equal(player.get('levelName'), 'Professional', 'new level is called professional');
});
```

## Testing Relationships

For relationships you probably only want to test that the relationship
declarations are setup properly.

Assume that a `User` can own a `Profile`.

> You can follow along by generating your own user and profile models with `ember
> generate model user` and `ember generate model profile`.

```javascript {data-filename=app/models/profile.js}
import DS from 'ember-data';

export default DS.Model.extend({
});
```

```javascript {data-filename=app/models/user.js}
import DS from 'ember-data';

export default DS.Model.extend({
  profile: DS.belongsTo('profile')
});
```

Then you could test that the relationship is wired up correctly
with this test.

```javascript {data-filename=tests/unit/models/user-test.js}
import { moduleForModel, test } from 'ember-qunit';
import Ember from 'ember';

moduleForModel('user', 'Unit | Model | user', {
  // Specify the other units that are required for this test.
  needs: ['model:profile']
});

test('should own a profile', function(assert) {
  const User = this.store().modelFor('user');
  const relationship = Ember.get(User, 'relationshipsByName').get('profile');

  assert.equal(relationship.key, 'profile', 'has relationship with profile');
  assert.equal(relationship.kind, 'belongsTo', 'kind of relationship is belongsTo');
});
```

_Ember Data contains extensive tests around the functionality of
relationships, so you probably don't need to duplicate those tests.  You could
look at the [Ember Data tests] for examples of deeper relationship testing if you
feel the need to do it._

[Ember Data]: https://github.com/emberjs/data
[Unit Testing Basics]: ../unit-testing-basics/
[Ember Data tests]: https://github.com/emberjs/data/tree/master/tests
