# Settings and Virtual Attributes in Rails Applications

---

@title[Application-Wide Settings]

Application-Wide Settings

---

* Application Name
* Contact Email
* Google Place ID for map on contact page

```ruby
create_table :application_settings do |t|
  t.string :name, null: false
  t.string :value, null: false
end

ApplicationSetting.create(name: 'contact_email', value: 'ulf@example.com')
```

---

@title[User-Specific Settings]

How about user specific settings?

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

User.first.view_settings.try(:[], :users) || 
  User::DEFAULTS[:users]
```

---

![](assets/images/sa-lists.png)

```ruby
class ViewCountSetting < ApplicationRecord
end

class User < ApplicationRecord
  DEFAULTS = {users: 10, posts: 30}

  has_many :view_count_settings
end

User.first.view_count_settings.find_by(name: :users)&.value || 
  User::DEFAULTS[:users]
```

---

Experimental or Temporary Features

---

> We'd like to keep track of user invitations for an upcoming campaign

> Let's try to restrict available reactions (😀☹️) for individual posts 

---

### Usually: The same options again.

* Add new columns to existing tables
* Create new model + table
* Start serializing
