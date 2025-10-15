# Exercise 2: "Making it Interactive with Turbo" - Live Updates & Modals

**What we're adding:** Modal editing, inline status toggling, live updates without page refresh - all with minimal JavaScript.

---

## Files to ADD or MODIFY (only ~150 lines of new code!)

### 1. Add toggle_active action to controller

**app/controllers/room_types_controller.rb** - Add one method:

```ruby
class RoomTypesController < ApplicationController
  before_action :set_room_type, only: %i[show edit update destroy toggle_active]

  # ... existing actions stay the same ...

  def toggle_active
    @room_type.update(active: !@room_type.active)
    respond_to do |format|
      format.turbo_stream
    end
  end

  private

  def set_room_type
    @room_type = RoomType.find(params[:id])
  end

  def room_type_params
    params.require(:room_type).permit(
      :name, :description, :base_price, :max_occupancy, :room_size,
      :bed_type, :view_type, :active
    )
  end
end
```

### 2. Add route for toggle action

**config/routes.rb**:

```ruby
Rails.application.routes.draw do
  resources :room_types do
    member do
      patch :toggle_active
    end
  end
  root "room_types#index"
end
```

### 3. Update index view with Turbo Frames

**app/views/room_types/index.html.erb** - Replace with:

```erb
<div class="mx-auto max-w-7xl px-4 py-8">
  <div class="flex justify-between items-center mb-8">
    <h1 class="text-3xl font-bold">Room Types</h1>
    <%= link_to "New Room Type", new_room_type_path, 
        data: { turbo_frame: "modal" },
        class: "bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700" %>
  </div>

  <%= turbo_frame_tag "modal" %>

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
          <%= turbo_frame_tag dom_id(room_type), data: { turbo_action: "advance" } do %>
            <tr class="hover:bg-gray-50">
              <td class="px-6 py-4">
                <div class="font-medium"><%= room_type.name %></div>
                <div class="text-sm text-gray-500"><%= truncate(room_type.description, length: 50) %></div>
              </td>
              <td class="px-6 py-4">
                <div class="font-semibold">$<%= number_with_precision(room_type.base_price, precision: 2) %></div>
              </td>
              <td class="px-6 py-4 text-sm">
                <%= room_type.max_occupancy %> guests • <%= room_type.room_size %>m² • <%= room_type.bed_type %>
              </td>
              <td class="px-6 py-4">
                <%= button_to toggle_active_room_type_path(room_type), 
                    method: :patch,
                    class: "px-2 py-1 text-xs rounded #{room_type.active? ? 'bg-green-100 text-green-800' : 'bg-gray-100 text-gray-800'}" do %>
                  <%= room_type.status_text %>
                <% end %>
              </td>
              <td class="px-6 py-4 text-right">
                <%= link_to "Edit", edit_room_type_path(room_type), 
                    data: { turbo_frame: "modal" },
                    class: "text-blue-600 hover:text-blue-800 mr-3" %>
                <%= button_to "Delete", room_type_path(room_type), 
                    method: :delete, 
                    class: "text-red-600 hover:text-red-800 inline", 
                    form: { data: { turbo_confirm: "Are you sure?" } } %>
              </td>
            </tr>
          <% end %>
        <% end %>
      </tbody>
    </table>
  </div>
</div>
```

### 4. Create modal form for new/edit

**app/views/room_types/new.html.erb** - Replace with:

```erb
<%= turbo_frame_tag "modal" do %>
  <div class="fixed inset-0 bg-gray-500 bg-opacity-75 flex items-center justify-center p-4 z-50">
    <div class="bg-white rounded-lg shadow-xl max-w-2xl w-full max-h-[90vh] overflow-y-auto">
      <div class="px-6 py-4 border-b">
        <h2 class="text-xl font-semibold">New Room Type</h2>
      </div>
      <div class="px-6 py-4">
        <%= render "form", room_type: @room_type %>
      </div>
    </div>
  </div>
<% end %>
```

**app/views/room_types/edit.html.erb** - Replace with:

```erb
<%= turbo_frame_tag "modal" do %>
  <div class="fixed inset-0 bg-gray-500 bg-opacity-75 flex items-center justify-center p-4 z-50">
    <div class="bg-white rounded-lg shadow-xl max-w-2xl w-full max-h-[90vh] overflow-y-auto">
      <div class="px-6 py-4 border-b">
        <h2 class="text-xl font-semibold">Edit Room Type</h2>
      </div>
      <div class="px-6 py-4">
        <%= render "form", room_type: @room_type %>
      </div>
    </div>
  </div>
<% end %>
```

### 5. Update the form to work with modals

**app/views/room_types/_form.html.erb** - Keep most of it, just update the submit button area:

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

### 6. Create Turbo Stream responses

**app/views/room_types/create.turbo_stream.erb** (NEW FILE):

```erb
<%= turbo_stream.prepend "room_types", partial: "room_type_row", locals: { room_type: @room_type } %>
<%= turbo_stream.update "modal", "" %>
```

**app/views/room_types/update.turbo_stream.erb** (NEW FILE):

```erb
<%= turbo_stream.replace dom_id(@room_type), partial: "room_type_row", locals: { room_type: @room_type } %>
<%= turbo_stream.update "modal", "" %>
```

**app/views/room_types/destroy.turbo_stream.erb** (NEW FILE):

```erb
<%= turbo_stream.remove dom_id(@room_type) %>
```

**app/views/room_types/toggle_active.turbo_stream.erb** (NEW FILE):

```erb
<%= turbo_stream.replace dom_id(@room_type), partial: "room_type_row", locals: { room_type: @room_type } %>
```

### 7. Create the reusable row partial

**app/views/room_types/_room_type_row.html.erb** (NEW FILE):

```erb
<%= turbo_frame_tag dom_id(room_type), data: { turbo_action: "advance" } do %>
  <tr class="hover:bg-gray-50">
    <td class="px-6 py-4">
      <div class="font-medium"><%= room_type.name %></div>
      <div class="text-sm text-gray-500"><%= truncate(room_type.description, length: 50) %></div>
    </td>
    <td class="px-6 py-4">
      <div class="font-semibold">$<%= number_with_precision(room_type.base_price, precision: 2) %></div>
    </td>
    <td class="px-6 py-4 text-sm">
      <%= room_type.max_occupancy %> guests • <%= room_type.room_size %>m² • <%= room_type.bed_type %>
    </td>
    <td class="px-6 py-4">
      <%= button_to toggle_active_room_type_path(room_type), 
          method: :patch,
          class: "px-2 py-1 text-xs rounded #{room_type.active? ? 'bg-green-100 text-green-800' : 'bg-gray-100 text-gray-800'}" do %>
        <%= room_type.status_text %>
      <% end %>
    </td>
    <td class="px-6 py-4 text-right">
      <%= link_to "Edit", edit_room_type_path(room_type), 
          data: { turbo_frame: "modal" },
          class: "text-blue-600 hover:text-blue-800 mr-3" %>
      <%= button_to "Delete", room_type_path(room_type), 
          method: :delete, 
          class: "text-red-600 hover:text-red-800 inline", 
          form: { data: { turbo_confirm: "Are you sure?" } } %>
    </td>
  </tr>
<% end %>
```

### 8. Update controller to handle Turbo Streams

**app/controllers/room_types_controller.rb** - Update create/update methods:

```ruby
class RoomTypesController < ApplicationController
  before_action :set_room_type, only: %i[show edit update destroy toggle_active]

  def index
    @room_types = RoomType.all.order(created_at: :desc)
  end

  def show
  end

  def new
    @room_type = RoomType.new
  end

  def edit
  end

  def create
    @room_type = RoomType.new(room_type_params)

    respond_to do |format|
      if @room_type.save
        format.html { redirect_to room_types_path, notice: "Room type was successfully created." }
        format.turbo_stream
      else
        format.html { render :new, status: :unprocessable_entity }
      end
    end
  end

  def update
    respond_to do |format|
      if @room_type.update(room_type_params)
        format.html { redirect_to room_types_path, notice: "Room type was successfully updated." }
        format.turbo_stream
      else
        format.html { render :edit, status: :unprocessable_entity }
      end
    end
  end

  def destroy
    @room_type.destroy!

    respond_to do |format|
      format.html { redirect_to room_types_path, notice: "Room type was successfully deleted." }
      format.turbo_stream
    end
  end

  def toggle_active
    @room_type.update(active: !@room_type.active)
    respond_to do |format|
      format.turbo_stream
    end
  end

  private

  def set_room_type
    @room_type = RoomType.find(params[:id])
  end

  def room_type_params
    params.require(:room_type).permit(
      :name, :description, :base_price, :max_occupancy, :room_size,
      :bed_type, :view_type, :active
    )
  end
end
```

### 9. Update index to have ID for prepending new items

**app/views/room_types/index.html.erb** - Add `id="room_types"` to tbody:

```erb
<div class="mx-auto max-w-7xl px-4 py-8">
  <div class="flex justify-between items-center mb-8">
    <h1 class="text-3xl font-bold">Room Types</h1>
    <%= link_to "New Room Type", new_room_type_path, 
        data: { turbo_frame: "modal" },
        class: "bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700" %>
  </div>

  <%= turbo_frame_tag "modal" %>

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
      <tbody class="divide-y" id="room_types">
        <% @room_types.each do |room_type| %>
          <%= render "room_type_row", room_type: room_type %>
        <% end %>
      </tbody>
    </table>
  </div>
</div>
```

---

## 🎉 EXERCISE 2 COMPLETE!

**What you added:** ~150 lines of code

**What you now have:**
- ✅ Modal editing (no page navigation!)
- ✅ Inline status toggling (click to toggle active/inactive)
- ✅ Live updates (new items appear instantly)
- ✅ Delete without reload
- ✅ All updates happen in real-time with NO custom JavaScript

**Demo this by:**
1. Click "New Room Type" - opens in modal
2. Submit form - modal closes, new row appears at top instantly
3. Click status badge - toggles active/inactive without reload
4. Click Edit - opens in modal, save updates the row in place
5. Delete - row disappears instantly

**The magic:** Turbo Frames + Turbo Streams handle ALL the interactivity. Zero JavaScript written!

Ready for Exercise 3 with Avo admin framework?
