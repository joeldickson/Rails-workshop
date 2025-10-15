You're absolutely right! I got carried away with the styling. Let me show you what **Rails actually generates** with minimal customization.

The scaffold command creates about **90% of the code automatically**. Here's what you *actually* get:

---

## Exercise 1: THE REAL "Zero to CRUD in 5 Minutes"

### Commands (this is literally all you type):

```bash
rails new agoda_extranet --css=tailwind --database=postgresql
cd agoda_extranet
rails db:create

rails generate scaffold RoomType \
  name:string \
  description:text \
  base_price:decimal \
  max_occupancy:integer \
  room_size:decimal \
  bed_type:string \
  view_type:string \
  active:boolean

rails db:migrate
rails db:seed  # optional
rails server
```

### What Rails GENERATES for you automatically:

**Model, Controller, Views, Routes** - all created by scaffold!

The only files you need to touch:

### 1. Add ONE method to the model (app/models/room_type.rb)
```ruby
class RoomType < ApplicationRecord
  validates :name, presence: true
  validates :base_price, presence: true, numericality: { greater_than: 0 }

  # Just this one method for display
  def status_text
    active? ? "Active" : "Inactive"
  end
end
```

### 2. Set the root route (config/routes.rb)
```ruby
Rails.application.routes.draw do
  resources :room_types
  root "room_types#index"  # Add this one line
end
```

### 3. Customize JUST the index view (app/views/room_types/index.html.erb)

Replace scaffold's table with this **~60 line version**:

```erb
<div class="mx-auto max-w-7xl px-4 py-8">
  <div class="flex justify-between items-center mb-8">
    <h1 class="text-3xl font-bold">Room Types</h1>
    <%= link_to "New Room Type", new_room_type_path, 
        class: "bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700" %>
  </div>

  <div class="bg-white shadow rounded-lg overflow-hidden">
    <table class="min-w-full">
      <thead class="bg-gray-50">
        <tr>
          <th class="px-6 py-3 text-left text-sm font-semibold">Name</th>
          <th class="px-6 py-3 text-left text-sm font-semibold">Price</th>
          <th class="px-6 py-3 text-left text-sm font-semibold">Details</th>
          <th class="px-6 py-3 text-left text-sm font-semibold">Status</th>
          <th class="px-6 py-3 text-right text-sm font-semibold">Actions</th>
        </tr>
      </thead>
      <tbody class="divide-y">
        <% @room_types.each do |room_type| %>
          <tr class="hover:bg-gray-50">
            <td class="px-6 py-4">
              <div class="font-medium"><%= room_type.name %></div>
              <div class="text-sm text-gray-500"><%= truncate(room_type.description, length: 50) %></div>
            </td>
            <td class="px-6 py-4">
              <div class="font-semibold">$<%= room_type.base_price %></div>
            </td>
            <td class="px-6 py-4 text-sm">
              <%= room_type.max_occupancy %> guests • <%= room_type.room_size %>m² • <%= room_type.bed_type %>
            </td>
            <td class="px-6 py-4">
              <span class="px-2 py-1 text-xs rounded <%= room_type.active? ? 'bg-green-100 text-green-800' : 'bg-gray-100' %>">
                <%= room_type.status_text %>
              </span>
            </td>
            <td class="px-6 py-4 text-right">
              <%= link_to "Edit", edit_room_type_path(room_type), class: "text-blue-600 hover:text-blue-800 mr-3" %>
              <%= button_to "Delete", room_type_path(room_type), method: :delete, 
                  class: "text-red-600 hover:text-red-800", 
                  form: { data: { turbo_confirm: "Are you sure?" } } %>
            </td>
          </tr>
        <% end %>
      </tbody>
    </table>
  </div>
</div>
```

### 4. Keep the form simple (app/views/room_types/_form.html.erb)

The scaffold generates this. Just keep it or make minimal tweaks:

```erb
<%= form_with(model: room_type) do |form| %>
  <% if room_type.errors.any? %>
    <div class="bg-red-50 p-4 mb-4 rounded">
      <h2 class="text-red-800 font-semibold"><%= pluralize(room_type.errors.count, "error") %></h2>
      <ul class="list-disc list-inside text-red-700">
        <% room_type.errors.each do |error| %>
          <li><%= error.full_message %></li>
        <% end %>
      </ul>
    </div>
  <% end %>

  <div class="space-y-4">
    <div>
      <%= form.label :name, class: "block font-medium mb-1" %>
      <%= form.text_field :name, class: "w-full border rounded px-3 py-2" %>
    </div>

    <div>
      <%= form.label :description, class: "block font-medium mb-1" %>
      <%= form.text_area :description, rows: 3, class: "w-full border rounded px-3 py-2" %>
    </div>

    <div class="grid grid-cols-2 gap-4">
      <div>
        <%= form.label :base_price, class: "block font-medium mb-1" %>
        <%= form.number_field :base_price, step: 0.01, class: "w-full border rounded px-3 py-2" %>
      </div>

      <div>
        <%= form.label :max_occupancy, class: "block font-medium mb-1" %>
        <%= form.number_field :max_occupancy, class: "w-full border rounded px-3 py-2" %>
      </div>
    </div>

    <div class="grid grid-cols-2 gap-4">
      <div>
        <%= form.label :room_size, "Room Size (m²)", class: "block font-medium mb-1" %>
        <%= form.number_field :room_size, step: 0.1, class: "w-full border rounded px-3 py-2" %>
      </div>

      <div>
        <%= form.label :bed_type, class: "block font-medium mb-1" %>
        <%= form.text_field :bed_type, class: "w-full border rounded px-3 py-2" %>
      </div>
    </div>

    <div>
      <%= form.label :view_type, class: "block font-medium mb-1" %>
      <%= form.text_field :view_type, class: "w-full border rounded px-3 py-2" %>
    </div>

    <div>
      <%= form.check_box :active, class: "mr-2" %>
      <%= form.label :active, "Active" %>
    </div>

    <div class="flex gap-2">
      <%= form.submit class: "bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700 cursor-pointer" %>
      <%= link_to "Cancel", room_types_path, class: "bg-gray-200 px-4 py-2 rounded hover:bg-gray-300" %>
    </div>
  </div>
<% end %>
```

---

## That's it! ~100 lines of custom code.

**Everything else was generated by Rails:**
- ✅ Controller with all 7 RESTful actions
- ✅ Model with database migration
- ✅ All view templates (index, show, new, edit)
- ✅ Routes configured
- ✅ Form handling with validations
- ✅ Full CRUD operations working

**The workshop point:**
"Look, I typed ONE command (`rails generate scaffold`), added some validations, and customized one view. That's 5 minutes of work for a fully functional CRUD app."

You're right - I way overcomplicated it. The power of Rails is that you DON'T need to write much code!
