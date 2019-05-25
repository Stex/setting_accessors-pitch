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
change_table :users do |t|
  t.text :view_settings
end

class User < ApplicationRecord
  serialize :view_settings
end

u = User.first
u.view_settings ||= {}
u.view_settings[:users] = 30
```
