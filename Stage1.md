Absolutely! Let's build a **Room Type Management System** for Agoda's extranet. This is more straightforward and perfect for demonstrating Rails' power.

---

## Exercise 1: "Zero to CRUD in 5 Minutes" - Room Type Management

**What we're building**: Hotel partners manage their room inventory (Deluxe Suite, Superior Room, etc.) with details like pricing, capacity, and amenities.

### Complete Setup

```bash
# Create new Rails 8 app
rails new agoda_extranet --css=tailwind --database=postgresql
cd agoda_extranet
rails db:create

# Generate the scaffold
rails generate scaffold RoomType \
  name:string \
  description:text \
  base_price:decimal \
  max_occupancy:integer \
  room_size:decimal \
  bed_type:string \
  view_type:string \
  floor_range:string \
  total_rooms:integer \
  available_rooms:integer \
  active:boolean

rails db:migrate
```

### db/seeds.rb
```ruby
# Clear existing data
RoomType.destroy_all

puts "Creating sample room types..."

room_types = [
  {
    name: 'Deluxe Ocean View Suite',
    description: 'Spacious suite with panoramic ocean views, king-size bed, separate living area, and premium amenities. Perfect for couples seeking luxury and romance.',
    base_price: 299.00,
    max_occupancy: 3,
    room_size: 45.0,
    bed_type: 'King Bed',
    view_type: 'Ocean View',
    floor_range: '15-20',
    total_rooms: 25,
    available_rooms: 18,
    active: true
  },
  {
    name: 'Superior City View Room',
    description: 'Modern room with city skyline views, comfortable queen bed, work desk, and complimentary Wi-Fi. Ideal for business travelers.',
    base_price: 189.00,
    max_occupancy: 2,
    room_size: 32.0,
    bed_type: 'Queen Bed',
    view_type: 'City View',
    floor_range: '10-14',
    total_rooms: 40,
    available_rooms: 32,
    active: true
  },
  {
    name: 'Standard Garden View Room',
    description: 'Cozy room overlooking tropical gardens, twin beds that can be combined, mini-fridge, and coffee maker. Great value for budget-conscious travelers.',
    base_price: 129.00,
    max_occupancy: 2,
    room_size: 28.0,
    bed_type: 'Twin Beds',
    view_type: 'Garden View',
    floor_range: '3-9',
    total_rooms: 60,
    available_rooms: 45,
    active: true
  },
  {
    name: 'Executive Suite',
    description: 'Premium suite with separate bedroom, dining area, executive lounge access, and concierge service. Designed for discerning travelers.',
    base_price: 449.00,
    max_occupancy: 4,
    room_size: 65.0,
    bed_type: '1 King + 1 Sofa Bed',
    view_type: 'Ocean View',
    floor_range: '18-22',
    total_rooms: 15,
    available_rooms: 8,
    active: true
  },
  {
    name: 'Family Room',
    description: 'Spacious family-friendly room with bunk beds, entertainment system, and connecting room option. Perfect for families with children.',
    base_price: 249.00,
    max_occupancy: 5,
    room_size: 42.0,
    bed_type: '1 Queen + Bunk Beds',
    view_type: 'Pool View',
    floor_range: '5-8',
    total_rooms: 20,
    available_rooms: 12,
    active: true
  },
  {
    name: 'Presidential Suite',
    description: 'Ultimate luxury with private terrace, jacuzzi, butler service, and panoramic views. The pinnacle of indulgence.',
    base_price: 899.00,
    max_occupancy: 4,
    room_size: 120.0,
    bed_type: '1 King + Study',
    view_type: 'Panoramic View',
    floor_range: '23',
    total_rooms: 2,
    available_rooms: 1,
    active: true
  },
  {
    name: 'Accessible Room',
    description: 'Fully accessible room with wheelchair-friendly features, roll-in shower, and support bars. Comfort for all guests.',
    base_price: 149.00,
    max_occupancy: 2,
    room_size: 35.0,
    bed_type: 'Queen Bed',
    view_type: 'Garden View',
    floor_range: '2',
    total_rooms: 8,
    available_rooms: 6,
    active: true
  },
  {
    name: 'Penthouse Loft (Coming Soon)',
    description: 'Modern loft-style penthouse currently under renovation. Opening Q4 2025.',
    base_price: 699.00,
    max_occupancy: 3,
    room_size: 85.0,
    bed_type: 'King Bed',
    view_type: 'Skyline View',
    floor_range: '24',
    total_rooms: 4,
    available_rooms: 0,
    active: false
  }
]

room_types.each do |room_type|
  RoomType.create!(room_type)
  puts "Created: #{room_type[:name]}"
end

puts "\n#{RoomType.count} room types created successfully!"
```

```bash
rails db:seed
```

### app/models/room_type.rb
```ruby
class RoomType < ApplicationRecord
  # Validations
  validates :name, presence: true, uniqueness: true
  validates :base_price, presence: true, numericality: { greater_than: 0 }
  validates :max_occupancy, presence: true, numericality: { greater_than: 0, only_integer: true }
  validates :room_size, presence: true, numericality: { greater_than: 0 }
  validates :total_rooms, presence: true, numericality: { greater_than_or_equal_to: 0, only_integer: true }
  validates :available_rooms, presence: true, numericality: { greater_than_or_equal_to: 0, only_integer: true }
  validate :available_rooms_not_greater_than_total

  # Scopes
  scope :active, -> { where(active: true) }
  scope :inactive, -> { where(active: false) }
  scope :by_price, -> { order(base_price: :asc) }
  scope :available, -> { where("available_rooms > 0") }

  # Constants
  BED_TYPES = [
    'King Bed',
    'Queen Bed',
    'Twin Beds',
    '2 Single Beds',
    '1 King + 1 Sofa Bed',
    '1 Queen + Bunk Beds',
    '1 King + Study'
  ].freeze

  VIEW_TYPES = [
    'Ocean View',
    'City View',
    'Garden View',
    'Pool View',
    'Mountain View',
    'Skyline View',
    'Panoramic View',
    'No View'
  ].freeze

  # Instance Methods
  def occupancy_rate
    return 0 if total_rooms.zero?
    ((total_rooms - available_rooms).to_f / total_rooms * 100).round(1)
  end

  def availability_status
    if available_rooms.zero?
      'sold_out'
    elsif occupancy_rate >= 80
      'limited'
    else
      'available'
    end
  end

  def status_badge_color
    return 'gray' unless active?
    
    case availability_status
    when 'sold_out' then 'red'
    when 'limited' then 'yellow'
    else 'green'
    end
  end

  def status_text
    return 'Inactive' unless active?
    
    case availability_status
    when 'sold_out' then 'Sold Out'
    when 'limited' then "Only #{available_rooms} left"
    else "#{available_rooms} available"
    end
  end

  private

  def available_rooms_not_greater_than_total
    if available_rooms && total_rooms && available_rooms > total_rooms
      errors.add(:available_rooms, "cannot be greater than total rooms")
    end
  end
end
```

### app/controllers/room_types_controller.rb
```ruby
class RoomTypesController < ApplicationController
  before_action :set_room_type, only: %i[show edit update destroy]

  def index
    @room_types = RoomType.all.order(base_price: :desc)
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

    if @room_type.save
      redirect_to room_types_path, notice: "Room type was successfully created."
    else
      render :new, status: :unprocessable_entity
    end
  end

  def update
    if @room_type.update(room_type_params)
      redirect_to room_types_path, notice: "Room type was successfully updated."
    else
      render :edit, status: :unprocessable_entity
    end
  end

  def destroy
    @room_type.destroy!
    redirect_to room_types_path, notice: "Room type was successfully deleted."
  end

  private

  def set_room_type
    @room_type = RoomType.find(params[:id])
  end

  def room_type_params
    params.require(:room_type).permit(
      :name, :description, :base_price, :max_occupancy, :room_size,
      :bed_type, :view_type, :floor_range, :total_rooms, :available_rooms, :active
    )
  end
end
```

### config/routes.rb
```ruby
Rails.application.routes.draw do
  resources :room_types
  root "room_types#index"
end
```

### app/views/layouts/application.html.erb
```erb
<!DOCTYPE html>
<html>
  <head>
    <title>Agoda Extranet - Partner Portal</title>
    <meta name="viewport" content="width=device-width,initial-scale=1">
    <%= csrf_meta_tags %>
    <%= csp_meta_tag %>
    <%= stylesheet_link_tag "tailwind", "inter-font", "data-turbo-track": "reload" %>
    <%= stylesheet_link_tag "application", "data-turbo-track": "reload" %>
    <%= javascript_importmap_tags %>
  </head>

  <body class="bg-gray-50">
    <!-- Header -->
    <nav class="bg-white shadow-sm border-b border-gray-200">
      <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div class="flex justify-between h-16">
          <div class="flex items-center">
            <div class="flex-shrink-0 flex items-center">
              <span class="text-2xl font-bold text-blue-600">agoda</span>
              <span class="ml-3 text-sm text-gray-500 border-l border-gray-300 pl-3">Partner Extranet</span>
            </div>
            <div class="hidden sm:ml-8 sm:flex sm:space-x-8">
              <%= link_to "Room Types", root_path, class: "border-blue-500 text-gray-900 inline-flex items-center px-1 pt-1 border-b-2 text-sm font-medium" %>
              <a href="#" class="border-transparent text-gray-500 hover:border-gray-300 hover:text-gray-700 inline-flex items-center px-1 pt-1 border-b-2 text-sm font-medium">Rate Plans</a>
              <a href="#" class="border-transparent text-gray-500 hover:border-gray-300 hover:text-gray-700 inline-flex items-center px-1 pt-1 border-b-2 text-sm font-medium">Reservations</a>
              <a href="#" class="border-transparent text-gray-500 hover:border-gray-300 hover:text-gray-700 inline-flex items-center px-1 pt-1 border-b-2 text-sm font-medium">Analytics</a>
            </div>
          </div>
          <div class="flex items-center">
            <span class="text-sm text-gray-700">Seaside Grand Hotel</span>
            <div class="ml-3 relative">
              <div class="h-8 w-8 rounded-full bg-blue-600 flex items-center justify-center text-white text-sm font-medium">
                SG
              </div>
            </div>
          </div>
        </div>
      </div>
    </nav>

    <!-- Flash Messages -->
    <% if notice %>
      <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 mt-4">
        <div class="rounded-md bg-green-50 p-4">
          <div class="flex">
            <div class="flex-shrink-0">
              <svg class="h-5 w-5 text-green-400" viewBox="0 0 20 20" fill="currentColor">
                <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z" clip-rule="evenodd"/>
              </svg>
            </div>
            <div class="ml-3">
              <p class="text-sm font-medium text-green-800"><%= notice %></p>
            </div>
          </div>
        </div>
      </div>
    <% end %>

    <!-- Main Content -->
    <main>
      <%= yield %>
    </main>

    <!-- Footer -->
    <footer class="bg-white border-t border-gray-200 mt-12">
      <div class="max-w-7xl mx-auto py-6 px-4 sm:px-6 lg:px-8">
        <p class="text-center text-sm text-gray-500">
          © 2025 Agoda Company Pte. Ltd. All Rights Reserved.
        </p>
      </div>
    </footer>
  </body>
</html>
```

### app/views/room_types/index.html.erb
```erb
<div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
  <!-- Header Section -->
  <div class="md:flex md:items-center md:justify-between mb-8">
    <div class="flex-1 min-w-0">
      <h1 class="text-3xl font-bold text-gray-900">Room Inventory</h1>
      <p class="mt-2 text-sm text-gray-700">
        Manage your property's room types and availability
      </p>
    </div>
    <div class="mt-4 flex md:mt-0 md:ml-4">
      <%= link_to new_room_type_path, class: "inline-flex items-center px-4 py-2 border border-transparent rounded-md shadow-sm text-sm font-medium text-white bg-blue-600 hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500" do %>
        <svg class="h-5 w-5 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v16m8-8H4"/>
        </svg>
        Add Room Type
      <% end %>
    </div>
  </div>

  <!-- Stats Cards -->
  <div class="grid grid-cols-1 gap-5 sm:grid-cols-2 lg:grid-cols-4 mb-8">
    <div class="bg-white overflow-hidden shadow rounded-lg">
      <div class="p-5">
        <div class="flex items-center">
          <div class="flex-shrink-0">
            <svg class="h-6 w-6 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 21V5a2 2 0 00-2-2H7a2 2 0 00-2 2v16m14 0h2m-2 0h-5m-9 0H3m2 0h5M9 7h1m-1 4h1m4-4h1m-1 4h1m-5 10v-5a1 1 0 011-1h2a1 1 0 011 1v5m-4 0h4"/>
            </svg>
          </div>
          <div class="ml-5 w-0 flex-1">
            <dl>
              <dt class="text-sm font-medium text-gray-500 truncate">Total Room Types</dt>
              <dd class="text-2xl font-semibold text-gray-900"><%= @room_types.count %></dd>
            </dl>
          </div>
        </div>
      </div>
    </div>

    <div class="bg-white overflow-hidden shadow rounded-lg">
      <div class="p-5">
        <div class="flex items-center">
          <div class="flex-shrink-0">
            <svg class="h-6 w-6 text-green-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z"/>
            </svg>
          </div>
          <div class="ml-5 w-0 flex-1">
            <dl>
              <dt class="text-sm font-medium text-gray-500 truncate">Active Types</dt>
              <dd class="text-2xl font-semibold text-gray-900"><%= @room_types.active.count %></dd>
            </dl>
          </div>
        </div>
      </div>
    </div>

    <div class="bg-white overflow-hidden shadow rounded-lg">
      <div class="p-5">
        <div class="flex items-center">
          <div class="flex-shrink-0">
            <svg class="h-6 w-6 text-blue-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 12l2-2m0 0l7-7 7 7M5 10v10a1 1 0 001 1h3m10-11l2 2m-2-2v10a1 1 0 01-1 1h-3m-6 0a1 1 0 001-1v-4a1 1 0 011-1h2a1 1 0 011 1v4a1 1 0 001 1m-6 0h6"/>
            </svg>
          </div>
          <div class="ml-5 w-0 flex-1">
            <dl>
              <dt class="text-sm font-medium text-gray-500 truncate">Total Inventory</dt>
              <dd class="text-2xl font-semibold text-gray-900"><%= @room_types.sum(:total_rooms) %></dd>
            </dl>
          </div>
        </div>
      </div>
    </div>

    <div class="bg-white overflow-hidden shadow rounded-lg">
      <div class="p-5">
        <div class="flex items-center">
          <div class="flex-shrink-0">
            <svg class="h-6 w-6 text-purple-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8c-1.657 0-3 .895-3 2s1.343 2 3 2 3 .895 3 2-1.343 2-3 2m0-8c1.11 0 2.08.402 2.599 1M12 8V7m0 1v8m0 0v1m0-1c-1.11 0-2.08-.402-2.599-1M21 12a9 9 0 11-18 0 9 9 0 0118 0z"/>
            </svg>
          </div>
          <div class="ml-5 w-0 flex-1">
            <dl>
              <dt class="text-sm font-medium text-gray-500 truncate">Avg. Rate</dt>
              <dd class="text-2xl font-semibold text-gray-900">
                $<%= number_with_precision(@room_types.average(:base_price) || 0, precision: 0) %>
              </dd>
            </dl>
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- Room Types Grid -->
  <div class="bg-white shadow overflow-hidden sm:rounded-lg">
    <ul role="list" class="divide-y divide-gray-200">
      <% @room_types.each do |room_type| %>
        <li class="hover:bg-gray-50 transition-colors duration-150">
          <div class="px-6 py-5">
            <div class="flex items-center justify-between">
              <div class="flex-1 min-w-0">
                <div class="flex items-center space-x-3">
                  <h3 class="text-lg font-medium text-gray-900 truncate">
                    <%= room_type.name %>
                  </h3>
                  <% color_classes = {
                    'green' => 'bg-green-100 text-green-800',
                    'yellow' => 'bg-yellow-100 text-yellow-800',
                    'red' => 'bg-red-100 text-red-800',
                    'gray' => 'bg-gray-100 text-gray-800'
                  } %>
                  <span class="inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium <%= color_classes[room_type.status_badge_color] %>">
                    <%= room_type.status_text %>
                  </span>
                </div>
                <p class="mt-1 text-sm text-gray-500 line-clamp-2">
                  <%= room_type.description %>
                </p>
                <div class="mt-3 flex items-center space-x-6 text-sm text-gray-500">
                  <div class="flex items-center">
                    <svg class="flex-shrink-0 mr-1.5 h-5 w-5 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17 20h5v-2a3 3 0 00-5.356-1.857M17 20H7m10 0v-2c0-.656-.126-1.283-.356-1.857M7 20H2v-2a3 3 0 015.356-1.857M7 20v-2c0-.656.126-1.283.356-1.857m0 0a5.002 5.002 0 019.288 0M15 7a3 3 0 11-6 0 3 3 0 016 0zm6 3a2 2 0 11-4 0 2 2 0 014 0zM7 10a2 2 0 11-4 0 2 2 0 014 0z"/>
                    </svg>
                    Up to <%= room_type.max_occupancy %> guests
                  </div>
                  <div class="flex items-center">
                    <svg class="flex-shrink-0 mr-1.5 h-5 w-5 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 8V4m0 0h4M4 4l5 5m11-1V4m0 0h-4m4 0l-5 5M4 16v4m0 0h4m-4 0l5-5m11 5l-5-5m5 5v-4m0 4h-4"/>
                    </svg>
                    <%= room_type.room_size %> m²
                  </div>
                  <div class="flex items-center">
                    <svg class="flex-shrink-0 mr-1.5 h-5 w-5 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 12h18M3 6h18M3 18h18"/>
                    </svg>
                    <%= room_type.bed_type %>
                  </div>
                  <div class="flex items-center">
                    <svg class="flex-shrink-0 mr-1.5 h-5 w-5 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z"/>
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M2.458 12C3.732 7.943 7.523 5 12 5c4.478 0 8.268 2.943 9.542 7-1.274 4.057-5.064 7-9.542 7-4.477 0-8.268-2.943-9.542-7z"/>
                    </svg>
                    <%= room_type.view_type %>
                  </div>
                </div>
              </div>
              <div class="ml-6 flex flex-col items-end space-y-3">
                <div class="text-right">
                  <div class="text-2xl font-bold text-gray-900">
                    $<%= number_with_precision(room_type.base_price, precision: 2) %>
                  </div>
                  <div class="text-xs text-gray-500">per night</div>
                </div>
                <div class="text-right text-sm">
                  <div class="text-gray-900 font-medium">
                    <%= room_type.available_rooms %>/<%= room_type.total_rooms %> available
                  </div>
                  <div class="text-gray-500">
                    <%= room_type.occupancy_rate %>% occupied
                  </div>
                </div>
                <div class="flex space-x-2">
                  <%= link_to room_type_path(room_type), class: "inline-flex items-center p-2 border border-gray-300 rounded-md shadow-sm text-sm font-medium text-gray-700 bg-white hover:bg-gray-50" do %>
                    <svg class="h-4 w-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z"/>
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M2.458 12C3.732 7.943 7.523 5 12 5c4.478 0 8.268 2.943 9.542 7-1.274 4.057-5.064 7-9.542 7-4.477 0-8.268-2.943-9.542-7z"/>
                    </svg>
                  <% end %>
                  <%= link_to edit_room_type_path(room_type), class: "inline-flex items-center p-2 border border-gray-300 rounded-md shadow-sm text-sm font-medium text-gray-700 bg-white hover:bg-gray-50" do %>
                    <svg class="h-4 w-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z"/>
                    </svg>
                  <% end %>
                  <%= button_to room_type_path(room_type), method: :delete, data: { turbo_confirm: "Are you sure you want to delete '#{room_type.name}'?" }, class: "inline-flex items-center p-2 border border-red-300 rounded-md shadow-sm text-sm font-medium text-red-700 bg-white hover:bg-red-50" do %>
                    <svg class="h-4 w-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"/>
                    </svg>
                  <% end %>
                </div>
              </div>
            </div>
          </div>
        </li>
      <% end %>
    </ul>

    <% if @room_types.empty? %>
      <div class="text-center py-12">
        <svg class="mx-auto h-12 w-12 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 21V5a2 2 0 00-2-2H7a2 2 0 00-2 2v16m14 0h2m-2 0h-5m-9 0H3m2 0h5M9 7h1m-1 4h1m4-4h1m-1 4h1m-5 10v-5a1 1 0 011-1h2a1 1 0 011 1v5m-4 0h4"/>
        </svg>
        <h3 class="mt-2 text-sm font-medium text-gray-900">No room types</h3>
        <p class="mt-1 text-sm text-gray-500">Get started by creating your first room type.</p>
        <div class="mt-6">
          <%= link_to new_room_type_path, class: "inline-flex items-center px-4 py-2 border border-transparent shadow-sm text-sm font-medium rounded-md text-white bg-blue-600 hover:bg-blue-700" do %>
            <svg class="-ml-1 mr-2 h-5 w-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v16m8-8H4"/>
            </svg>
            Add Room Type
          <% end %>
        </div>
      </div>
    <% end %>
  </div>
</div>
```

### app/views/room_types/_form.html.erb
```erb
<%= form_with(model: room_type, class: "space-y-8") do |form| %>
  <% if room_type.errors.any? %>
    <div class="rounded-md bg-red-50 p-4">
      <div class="flex">
        <div class="flex-shrink-0">
          <svg class="h-5 w-5 text-red-400" viewBox="0 0 20 20" fill="currentColor">
            <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z" clip-rule="evenodd"/>
          </svg>
        </div>
        <div class="ml-3">
          <h3 class="text-sm font-medium text-red-800">
            <%= pluralize(room_type.errors.count, "error") %> prohibited this room type from being saved:
          </h3>
          <div class="mt-2 text-sm text-red-700">
            <ul class="list-disc pl-5 space-y-1">
              <% room_type.errors.each do |error| %>
                <li><%= error.full_message %></li>
              <% end %>
            </ul>
          </div>
        </div>
      </div>
    </div>
  <% end %>

  <!-- Basic Information -->
  <div class="bg-white shadow sm:rounded-lg">
    <div class="px-4 py-5 sm:p-6">
      <h3 class="text-lg font-medium leading-6 text-gray-900 mb-4">Basic Information</h3>
      <div class="grid grid-cols-1 gap-y-6 gap-x-4 sm:grid-cols-6">
        <div class="sm:col-span-4">
          <%= form.label :name, class: "block text-sm font-medium text-gray-700" %>
          <%= form.text_field :name, placeholder: "e.g., Deluxe Ocean View Suite", class: "mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-blue-500 focus:ring-blue-500 sm:text-sm" %>
        </div>

        <div class="sm:col-span-2">
          <div class="flex items-center h-full">
            <%= form.check_box :active, class: "h-4 w-4 rounded border-gray-300 text-blue-600 focus:ring-blue-500" %>
            <%= form.label :active, "Active listing", class: "ml-2 block text-sm font-medium text-gray-700" %>
          </div>
        </div>

        <div class="sm:col-span-6">
          <%= form.label :description, class: "block text-sm font-medium text-gray-700" %>
          <%= form.text_area :description, rows: 3, placeholder: "Describe what makes this room special...", class: "mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-blue-500 focus:ring-blue-500 sm:text-sm" %>
          <p class="mt-1 text-xs text-gray-500">This description will be shown to guests on Agoda</p>
        </div>
      </div>
    </div>
  </div>

  <!-- Room Details -->
  <div class="bg-white shadow sm:rounded-lg">
    <div class="px-4 py-5 sm:p-6">
      <h3 class="text-lg font-medium leading-6 text-gray-900 mb-4">Room Details</h3>
      <div class="grid grid-cols-1 gap-y-6 gap-x-4 sm:grid-cols-6">
        <div class="sm:col-span-2">
          <%= form.label :bed_type, class: "block text-sm font-medium text-gray-700" %>
          <%= form.select :bed_type, RoomType::BED_TYPES, { include_blank: "Select bed type" }, class: "mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-blue-500 focus:ring-blue-500 sm:text-sm" %>
        </div>

        <div class="sm:col-span-2">
          <%= form.label :view_type, class: "block text-sm font-medium text-gray-700" %>
          <%= form.select :view_type, RoomType::VIEW_TYPES, { include_blank: "Select view type" }, class: "mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-blue-500 focus:ring-blue-500 sm:text-sm" %>
        </div>

        <div class="sm:col-span-2">
          <%= form.label :floor_range, class: "block text-sm font-medium text-gray-700" %>
          <%= form.text_field :floor_range, placeholder: "e.g., 10-15", class: "mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-blue-500 focus:ring-blue-500 sm:text-sm" %>
        </div>

        <div class="sm:col-span-2">
          <%= form.label :max_occupancy, "Maximum Occupancy", class: "block text-sm font-medium text-gray-700" %>
          <%= form.number_field :max_occupancy, min: 1, class: "mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-blue-500 focus:ring-blue-500 sm:text-sm" %>
          <p class="mt-1 text-xs text-gray-500">Maximum number of guests</p>
        </div>

        <div class="sm:col-span-2">
          <%= form.label :room_size, "Room Size (m²)", class: "block text-sm font-medium text-gray-700" %>
          <%= form.number_field :room_size, step: 0.1, class: "mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-blue-500 focus:ring-blue-500 sm:text-sm" %>
        </div>

        <div class="sm:col-span-2">
          <%= form.label :base_price, "Base Price (USD)", class: "block text-sm font-medium text-gray-700" %>
          <div class="mt-1 relative rounded-md shadow-sm">
            <div class="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none">
              <span class="text-gray-500 sm:text-sm">$</span>
            </div>
            <%= form.number_field :base_price, step: 0.01, class: "pl-7 block w-full rounded-md border-gray-300 focus:border-blue-500 focus:ring-blue-500 sm:text-sm" %>
          </div>
          <p class="mt-1 text-xs text-gray-500">Price per night before taxes</p>
        </div>
      </div>
    </div>
  </div>

  <!-- Inventory Management -->
  <div class="bg-white shadow sm:rounded-lg">
    <div class="px-4 py-5 sm:p-6">
      <h3 class="text-lg font-medium leading-6 text-gray-900 mb-4">Inventory Management</h3>
      <div class="grid grid-cols-1 gap-y-6 gap-x-4 sm:grid-cols-6">
        <div class="sm:col-span-3">
          <%= form.label :total_rooms, "Total Rooms", class: "block text-sm font-medium text-gray-700" %>
          <%= form.number_field :total_rooms, min: 0, class: "mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-blue-500 focus:ring-blue-500 sm:text-sm" %>
          <p class="mt-1 text-xs text-gray-500">Total number of rooms of this type</p>
        </div>

        <div class="sm:col-span-3">
          <%= form.label :available_rooms, "Currently Available", class: "block text-sm font-medium text-gray-700" %>
          <%= form.number_field :available_rooms, min: 0, class: "mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-blue-500 focus:ring-blue-500 sm:text-sm" %>
          <p class="mt-1 text-xs text-gray-500">Rooms available for booking</p>
        </div>
      </div>
    </div>
  </div>

  <!-- Form Actions -->
  <div class="flex justify-end space-x-3">
    <%= link_to "Cancel", room_types_path, class: "px-4 py-2 border border-gray-300 rounded-md shadow-sm text-sm font-medium text-gray-700 bg-white hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500" %>
    <%= form.submit class: "px-4 py-2 border border-transparent rounded-md shadow-sm text-sm font-medium text-white bg-blue-600 hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500" %>
  </div>
<% end %>
```

### app/views/room_types/new.html.erb
```erb
<div class="max-w-4xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
  <div class="mb-8">
    <h1 class="text-3xl font-bold text-gray-900">Add New Room Type</h1>
    <p class="mt-2 text-sm text-gray-700">
      Fill in the details for your new room type
    </p>
  </div>

  <%= render "form", room_type: @room_type %>
</div>
```

### app/views/room_types/edit.html.erb
```erb
<div class="max-w-4xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
  <div class="mb-8">
    <h1 class="text-3xl font-bold text-gray-900">Edit Room Type</h1>
    <p class="mt-2 text-sm text-gray-700">
      Update the details for <%= @room_type.name %>
    </p>
  </div>

  <%= render "form", room_type: @room_type %>
</div>
```

### app/views/room_types/show.html.erb
```erb
<div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
  <div class="mb-8">
    <%= link_to room_types_path, class: "text-sm font-medium text-blue-600 hover:text-blue-800" do %>
      ← Back to Room Types
    <% end %>
  </div>

  <div class="bg-white shadow overflow-hidden sm:rounded-lg">
    <div class="px-4 py-5 sm:px-6 flex justify-between items-center">
      <div>
        <h1 class="text-2xl font-bold text-gray-900"><%= @room_type.name %></h1>
        <p class="mt-1 max-w-2xl text-sm text-gray-500"><%= @room_type.description %></p>
      </div>
      <div class="flex space-x-3">
        <%= link_to edit_room_type_path(@room_type), class: "inline-flex items-center px-4 py-2 border border-gray-300 rounded-md shadow-sm text-sm font-medium text-gray-700 bg-white hover:bg-gray-50" do %>
          <svg class="h-4 w-4 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z"/>
          </svg>
          Edit
        <% end %>
        <%= button_to room_type_path(@room_type), method: :delete, data: { turbo_confirm: "Are you sure?" }, class: "inline-flex items-center px-4 py-2 border border-red-300 rounded-md shadow-sm text-sm font-medium text-red-700 bg-white hover:bg-red-50" do %>
          <svg class="h-4 w-4 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"/>
          </svg>
          Delete
        <% end %>
      </div>
    </div>
    <div class="border-t border-gray-200 px-4 py-5 sm:p-0">
      <dl class="sm:divide-y sm:divide-gray-200">
        <div class="py-4 sm:py-5 sm:grid sm:grid-cols-3 sm:gap-4 sm:px-6">
          <dt class="text-sm font-medium text-gray-500">Base Price</dt>
          <dd class="mt-1 text-sm text-gray-900 sm:mt-0 sm:col-span-2">
            <span class="text-2xl font-bold">${%= number_with_precision(@room_type.base_price, precision: 2) %></span>
            <span class="text-gray-500">per night</span>
          </dd>
        </div>
        <div class="py-4 sm:py-5 sm:grid sm:grid-cols-3 sm:gap-4 sm:px-6">
          <dt class="text-sm font-medium text-gray-500">Maximum Occupancy</dt>
          <dd class="mt-1 text-sm text-gray-900 sm:mt-0 sm:col-span-2"><%= @room_type.max_occupancy %> guests</dd>
        </div>
        <div class="py-4 sm:py-5 sm:grid sm:grid-cols-3 sm:gap-4 sm:px-6">
          <dt class="text-sm font-medium text-gray-500">Room Size</dt>
          <dd class="mt-1 text-sm text-gray-900 sm:mt-0 sm:col-span-2"><%= @room_type.room_size %> m²</dd>
        </div>
        <div class="py-4 sm:py-5 sm:grid sm:grid-cols-3 sm:gap-4 sm:px-6">
          <dt class="text-sm font-medium text-gray-500">Bed Type</dt>
          <dd class="mt-1 text-sm text-gray-900 sm:mt-0 sm:col-span-2"><%= @room_type.bed_type %></dd>
        </div>
        <div class="py-4 sm:py-5 sm:grid sm:grid-cols-3 sm:gap-4 sm:px-6">
          <dt class="text-sm font-medium text-gray-500">View Type</dt>
          <dd class="mt-1 text-sm text-gray-900 sm:mt-0 sm:col-span-2"><%= @room_type.view_type %></dd>
        </div>
        <div class="py-4 sm:py-5 sm:grid sm:grid-cols-3 sm:gap-4 sm:px-6">
          <dt class="text-sm font-medium text-gray-500">Floor Range</dt>
          <dd class="mt-1 text-sm text-gray-900 sm:mt-0 sm:col-span-2"><%= @room_type.floor_range %></dd>
        </div>
        <div class="py-4 sm:py-5 sm:grid sm:grid-cols-3 sm:gap-4 sm:px-6">
          <dt class="text-sm font-medium text-gray-500">Inventory</dt>
          <dd class="mt-1 text-sm text-gray-900 sm:mt-0 sm:col-span-2">
            <div class="flex items-center space-x-4">
              <span><%= @room_type.available_rooms %> available of <%= @room_type.total_rooms %> total</span>
              <span class="px-2 py-1 text-xs font-semibold rounded-full bg-gray-100 text-gray-800">
                <%= @room_type.occupancy_rate %>% occupied
              </span>
            </div>
          </dd>
        </div>
        <div class="py-4 sm:py-5 sm:grid sm:grid-cols-3 sm:gap-4 sm:px-6">
          <dt class="text-sm font-medium text-gray-500">Status</dt>
          <dd class="mt-1 text-sm text-gray-900 sm:mt-0 sm:col-span-2">
            <% if @room_type.active %>
              <span class="inline-flex items-center px-3 py-1 rounded-full text-sm font-medium bg-green-100 text-green-800">
                Active
              </span>
            <% else %>
              <span class="inline-flex items-center px-3 py-1 rounded-full text-sm font-medium bg-gray-100 text-gray-800">
                Inactive
              </span>
            <% end %>
          </dd>
        </div>
      </dl>
    </div>
  </div>
</div>
```

---

**🎉 EXERCISE 1 COMPLETE!**

**What you've built:**
- Full CRUD interface for room types
- Professional Agoda-style UI
- Stats dashboard
- Form validation
- ~600 lines of code total (mostly Tailwind styling)

**Time to build:** ~5 minutes after running scaffold!

**Demo this by:**
1. Running `rails server`
2. Visit http://localhost:3000
3. Show how scaffold generated everything
4. Create/Edit/Delete a room type
5. Show the beautiful UI with minimal custom code

---

Ready for **Exercise 2** where we add Turbo for live updates, modal editing, and instant status toggling?
