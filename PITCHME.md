# Settings and Virtual Attributes in Rails Applications

---

## Reoccurring Problems

---

@title[Application-Wide Settings]

#### 1. Application-Wide Settings

---

* Application Name
* Contact Email
* Google Place ID for map on contact page
* The current meaning of meaning of life, the universe and everything

---

* Application Name
* Contact Email
* Google Place ID for map on contact page
* The current meaning of meaning of life, the universe and everything

```ruby
create_table :application_settings do |t|
  t.string :name, null: false
  t.string :value, null: false
end

ApplicationSetting.create(name: 'contact_email', 
                          value: 'ulf@example.com')
```

---

@title[User-Specific Settings]

#### 2. User-Specific Settings

---

![](assets/images/user-list.png)

---

![](assets/images/user-list.png)

```ruby
change_table :users do |t|
  t.integer :users_per_page, default: 10
end
```

---

![](assets/images/sa-lists.png)

```ruby
change_table :users do |t|
  t.integer :users_per_page, default: 10
  t.integer :posts_per_page, default: 30
end
```

---

![](assets/images/sa-lists.png)

```ruby
class User < ApplicationRecord
  DEFAULTS = {users: 10, posts: 30}

  # t.text :view_settings
  serialize :view_settings
end
```

```ruby
User.first.view_settings.try(:[], :users) || 
  User::DEFAULTS[:users]
```

---

![](assets/images/sa-lists.png)

```ruby
class ViewCountSetting < ApplicationRecord
end
```

```ruby
class User < ApplicationRecord
  DEFAULTS = {users: 10, posts: 30}

  has_many :view_count_settings
end
```

```ruby
User.first.view_count_settings.find_by(name: :users)&.value || 
  User::DEFAULTS[:users]
```

---

![](assets/images/spinthink.gif)

Do I really want to pollute my database schema with *that* specific columns/tables?

---

#### 3. Experimental or Temporary Features

---

@snap[north-west span-50]
![](assets/images/manager1.png)
@snapend

@snap[north-east span-50]
@box[bg-green rounded]("We need to keep track of user invitations for an upcoming campaign")
@snapend

@snap[south-west span-50]
@box[bg-gold text-white rounded]("We want to test restricting allowed reactions for certain posts")
@snapend

@snap[south-east span-50]
![](assets/images/manager2.png)
@snapend

---

#### The same options again.

* Add new columns to existing tables
* Create new model + table
* Start serializing

---

![](assets/images/ThonkSpin.gif)

Why should I write migrations only to revert them a week later?

---

#### What I wanted

* All of the previous solutions
* Not having to create single-purpose tables/columns
* Not having to change my database schema 

---

## `setting_accessors`

---

#### Global Key-Value-Store

```ruby
Setting.the_meaning_of_life ||= 42
Setting.the_meaning_of_life #=> 42
```

```ruby
Setting[:meaning_of_life] ||= 42
Setting[:meaning_of_life] #=> 42
```

```ruby
Setting.set(:meaning_of_life, 42)
Setting.get(:meaning_of_life) #=> 42
```

---

#### Assigned Records

```ruby
universe = Universe.first
Setting.set(:meaning_of_life, 43, assignable: universe)
Setting.get(:the_meaning_of_life, universe) #=> 43
```

```ruby
Setting[:the_meaning_of_life, universe] = 43
Setting.the_meaning_of_life(universe) #=> 43
```

---

#### Assigned Records

```ruby
def action_key
  [controller_path, action_name].join('_')
end
```

```ruby
def items_per_page
  Setting.get(action_key, current_user) || 30
end
```

```ruby
def set_items_per_page
  Setting.set(action_key, 
              params[:per_page], 
              assignable: current_user)
end
```

---

#### Virtual Attributes

```ruby
class User < ApplicationRecord
  setting_accessor :invited_users, type: :integer, default: 0
end
```

```ruby
u = User.first
u.invited_users #=> 0
u.update_attributes(invited_users: 1)
```

---

#### Virtual Attributes

```ruby
class Post < ApplicationRecord
  setting_accessor :allowed_reactions, 
                   type:    :polymorphic, 
                   default: ['happy', 'sad', 'thinking'] 
end
```

```ruby
p = Post.first
p.allowed_reactions #=> ['happy', 'sad', 'thinking']
p.allowed_reactions -= ['thinking']
p.save
```

---

#### Virtual Attributes

**Validations**

```ruby
class User < ApplicationRecord
  setting_accessor :invited_users, type: :integer, default: 0
  
  validates :invited_users,
            numericality: {less_than_or_equal_to: 100}
end
```

---

#### Virtual Attributes

**Helper Methods**

```ruby
class User < ApplicationRecord
  setting_accessor :invited_users, type: :integer, default: 0
end
```

```ruby
u = User.first
u.invited_users = '5'
u.invited_users #=> 5 (Already typecasted)

u.changed? #=> true
u.invited_users_changed? #=> true
u.invited_users_was #=> 0
u.invited_users_before_type_cast #=> '5'

u.reload
u.changed? #=> false
```
