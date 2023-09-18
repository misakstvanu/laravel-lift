I love Lift, it's great. That said, it took me a lot of customization to integrate it to my projects without conflicts with other Laravel features. I would like to talk a little about it so I can maybe contribute the wanted changes or create some kind of Lift Integrator for the unwanted.

I would like to share the problems and some of the solutions here and maybe something good can come out of it...

Give me a moment, I will post my thoughts in separate comments.

##  $attributes and properties out of sync

1. When setting a property like so `$entity->phone_number` Laravel's isDirty(), getDirty(), getAttribute()... are unusable, since they work with $attributes, that are not changed by this.
2. When filling (or updating...) with `$entity->fill(['phone_number' => 123456789])`, those methods work, but it's in turn not possible to access current data by object's properties.

### Fix
I don't think there is really a good fix. Just to commit to one style and be careful. Or customize the affected methods. What I did tho is customize `->fill()` method to affect properties also. But it doesn't solve the underlying problem.

## Observers
There is a lot of incompatibilities coming from tyhe nature of Lift.
1. If your observer runs after Lift, any changes made to properties will not be preserved in database.
2. If your observer runs before Lift, model's properties might not be initialized (retrieving, retrieved) or properties might not be up to date (creating, saving after `-> update()`)

### Fix
I fixed it by registering Lift, then observers, then a last observer to sync properties and $attributes.
```
//simplification
class Model {
    use Lift;
    use Observers;
    use SyncLiftAttributes;
}
```


## Lift initialization listeners
1. When replicating (`->replicate()`) a model, Lift does not run and object's properties are not initialized.
2. Same goes for observers listening to `created` event.

### My fix
Add more listeners to bootLift. E.g.: 
```
        static::replicating(function (Model $model) {
            self::fillProperties($model);

            foreach ($model->dispatchEvents as $prop) {
                $event = self::watchedProperties()[$prop];
                event(new $event($model));
            }

            $model->dispatchEvents = [];
        });
```
There might be potential side effects on duplicating dispatched events and performance (reruns) I didn't investigate.
