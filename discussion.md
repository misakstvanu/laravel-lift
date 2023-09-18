I love Lift, it's great. That said, it took me a lot of customization to integrate it to my projects without conflicts with other Laravel features. 

I would like to share the problems and some of the solutions here and maybe something good can come out of it...

Give me a moment, I will post my thoughts in separate comments.

##  $attributes and properties out of sync

1. When setting a property like so `$entity->phone_number` Laravel's isDirty(), getDirty(), getAttribute()... are unusable, since they work with $attributes, that are not changed by this.
2. When filling (or updating...) with `$entity->fill(['phone_number' => 123456789])`, 

