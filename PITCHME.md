# Settings and Virtual Attributes in Rails Applications

---

## Application-wide Settings

Should be changeable without having to deploy:

* Application Name
* Contact Email
* Google Place ID for map on contact page

---

## User-specific Settings

Example: "How many items would you like to see in the user list?"

![](assets/images/user-list.png)´

```ruby
change_table :users do |t|
  t.integer :users_per_page, default: 10
end
```

---

![](assets/images/posts-list.png)´

