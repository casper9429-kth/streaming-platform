# One-Day Ruby on Rails Crash Course Plan (for LiveKit Project)

This plan focuses on getting you from zero Rails knowledge to understanding the core concepts needed to start building your LiveKit application within an ~8-hour workday. It emphasizes "learn by doing" and leverages your existing LiveKit knowledge.

**Philosophy:** Use generators for speed, understand the output, focus on the "happy path", get immediate feedback, and be aware of key tools.

**Prerequisites:**
* Basic web concepts (HTTP, HTML, CSS, JS).
* Command line/terminal comfort.
* Text editor (like VS Code).
* Ruby, Rails (`gem install rails`), and a database (SQLite default is fine) installed.
* Your LiveKit proficiency.

---

## Before You Start (Prep)

* **Verify Installations:** Check `ruby -v` and `rails -v`.
* **Goal Reminder:** Keep the LiveKit integration goal in mind.

---

## Hour 1: Setup, Structure & Key Tools (Approx. 60 mins)

1.  **Install Ruby & Rails:**
    * Use `rbenv`, `RVM` (Mac/Linux) or `RubyInstaller w/DevKit` (Windows).
    * Verify versions.
2.  **Create App & Understand Gems:**
    * `rails new livekit_app` (Uses SQLite by default).
    * `cd livekit_app`
    * **`Gemfile`:** Open it. This lists project dependencies (gems). Note the `rails` gem.
    * **`bundle install`:** Run this. It installs gems from `Gemfile`. (`rails new` usually runs it initially).
3.  **Explore Core Structure:**
    * Briefly look at:
        * `app/`: Models (`app/models`), Views (`app/views`), Controllers (`app/controllers`).
        * `config/`: Configuration files (`routes.rb`, `database.yml`).
        * `db/`: Database schema (`schema.rb`), migrations (`db/migrate`).
4.  **Introduce Key Tools:**
    * `bin/rails server` (or `rails s`): Starts the development server. **Start it now.**
    * `bin/rails console` (or `rails c`): Interactive console for experimenting.
    * `bin/rails routes`: Lists all defined URL routes. **Run it now.**
    * `bin/rails generate` (or `rails g`): Creates code skeletons.
5.  **See it Live:**
    * Visit `http://localhost:3000` in your browser. See the default Rails page.

**Goal:** Get the environment running, understand key directories, and learn essential Rails commands.

---

## Hour 2: Routing, Controller, View (The Request Cycle) (Approx. 60 mins)

1.  **Concept:** Trace the path: Browser Request -> `config/routes.rb` -> Controller#action -> View (`.html.erb`) -> Browser Response.
2.  **Generate Controller & View:**
    * `bin/rails generate controller Pages home --skip-routes`
3.  **Define the Root Route:**
    * Open `config/routes.rb`. Add `root "pages#home"`.
    * Run `bin/rails routes` again to see the new route.
4.  **Add Basic Content:**
    * Edit `app/views/pages/home.html.erb`. Add `<h1>LiveKit Streaming!</h1>`.
    * Refresh `http://localhost:3000`.
5.  **Introduce ERB (Embedded Ruby):**
    * In `app/controllers/pages_controller.rb`, inside the `home` method, add:
        ```ruby
        def home
          @time = Time.now
        end
        ```
    * In `app/views/pages/home.html.erb`, add:
        ```html
        <p>The time is: <%= @time %></p>
        ```
    * Refresh the page. Understand `@instance_variables` pass data, `<%= %>` executes & displays Ruby.
    * **Where to look:** Rails Guides: "Rails Routing from the Outside In", "Action Controller Overview", "Action View Overview".

**Goal:** Understand the MVC request flow, map a URL, render a view, and display basic dynamic content.

---

## Hour 3: Model, Migration, Console CRUD (Approx. 60 mins)

1.  **Concept:** Active Record (ORM). Model = Ruby object representing DB table row. Migration = Instructions to change DB schema.
2.  **Generate Stream Model & Migration:**
    * `bin/rails generate model Stream name:string status:string`
    * **Analyze:** Look at `app/models/stream.rb`. Look at the timestamped file in `db/migrate/`.
3.  **Run Migration:**
    * `bin/rails db:migrate` (Creates/updates the `streams` table in `db/development.sqlite3`).
4.  **Experiment in Console:**
    * `bin/rails console`
    * Try commands:
        ```ruby
        Stream.create(name: "Test Stream 1", status: "pending")
        Stream.create(name: "Test Stream 2", status: "live")
        Stream.all
        my_stream = Stream.find_by(name: "Test Stream 1")
        my_stream.update(status: "finished")
        my_stream.destroy
        exit
        ```
    * **Where to look:** Rails Guides: "Active Record Basics", "Active Record Migrations".

**Goal:** Understand models, migrations, and perform basic database operations (CRUD) via the console.

---

## Hour 4: Scaffold & Analyzing Generated Code (Approx. 60 mins)

1.  **Concept:** `scaffold` builds *everything* (Model, Migration, Controller, Views, Routes) for standard CRUD. Fast, but generates lots of code to understand.
2.  **Generate Scaffold:**
    * `bin/rails g scaffold Stream name:string status:string` (`g` = shorthand for `generate`).
3.  **Migrate:**
    * `bin/rails db:migrate`
4.  **Analyze the Output (Crucial Step):**
    * **Routes:** Run `bin/rails routes`. Note the `resources :streams` line and all the generated routes (index, show, new, create, edit, update, destroy).
    * **Controller:** Open `app/controllers/streams_controller.rb`. Read actions (e.g., `index`, `show`, `create`). Note `@stream = Stream.find(params[:id])` and `stream_params` (Strong Parameters for security).
    * **Views:** Look in `app/views/streams/`.
        * `index.html.erb`: Note iteration (`<% @streams.each do |stream| %>`) and helpers (`link_to`, `stream_path(stream)`).
        * `_form.html.erb`: Note `form_with(model: stream)` helper and form fields. This is the standard Rails form builder.
        * `show.html.erb`: Displays single stream details.
    * **Layout:** Briefly look at `app/views/layouts/application.html.erb`. The main site template (`yield` inserts the specific view).
5.  **Test in Browser:**
    * Restart server (`rails s`). Go to `http://localhost:3000/streams`.
    * Click "New Stream", create one. Click Edit, Show, Destroy links. See it work.

**Goal:** Use scaffold for speed, but *understand* the generated CRUD code structure (routes, controller actions, views, forms, layouts).

---

## Hour 5: User Authentication with Devise (Approx. 75 mins - *May need buffer*)

1.  **Concept:** Need users. Devise gem is the standard, quick way.
2.  **Add & Setup Devise:**
    * Add `gem 'devise'` to `Gemfile`.
    * `bundle install`.
    * `bin/rails generate devise:install`
        * **Action Required:** Follow output instructions. Usually need to set default URL options for the mailer in `config/environments/development.rb`. Add: `config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }`
    * `bin/rails generate devise User` (Creates User model and migration).
    * `bin/rails db:migrate`
3.  **Protect Streams:**
    * Add `before_action :authenticate_user!` to the top of `app/controllers/streams_controller.rb`.
4.  **Test Authentication Flow:**
    * Restart `rails s`.
    * Try accessing `/streams` -> Redirected to `/users/sign_in`.
    * Go to `/users/sign_up`. Create an account.
    * Log in. Now `/streams` should be accessible.
    * Note: Devise provides helpers like `current_user` and `user_signed_in?`.
    * **Where to look:** Devise Gem documentation (GitHub), GoRails Devise tutorials.

**Goal:** Implement basic user sign-up/login using Devise and protect a resource.

---

## Hour 6: Associations & Linking Data (Approx. 45 mins)

1.  **Concept:** Connect models: `User has_many :streams`, `Stream belongs_to :user`.
2.  **Generate Migration for Link:**
    * `bin/rails generate migration AddUserRefToStreams user:references` (Adds `user_id` column to `streams` table).
3.  **Migrate:**
    * `bin/rails db:migrate`
4.  **Define Associations in Models:**
    * `app/models/user.rb`: add `has_many :streams`
    * `app/models/stream.rb`: add `belongs_to :user`
5.  **Update Controller Logic (Scope Data to User):**
    * `app/controllers/streams_controller.rb`:
        * `index` action: Change `@streams = Stream.all` to `@streams = current_user.streams`
        * `create` action: Before `@stream.save`, add the line `@stream.user = current_user`
        * (Optional but recommended): Update `show`, `edit`, `update`, `destroy` actions to find streams via `current_user.streams.find(params[:id])` instead of `Stream.find(params[:id])` for security.
6.  **Test:**
    * Create a new stream while logged in. Verify it's associated with your user (e.g., check `Stream.last.user` in `rails c`).
    * The `/streams` index page should now only show streams belonging to the logged-in user.
    * **Where to look:** Rails Guides: "Active Record Associations".

**Goal:** Connect two models relationally (one-to-many) and update controller logic to respect data ownership.

---

## Hour 7: LiveKit Server SDK & Token Endpoint (Approx. 60 mins - *May need buffer*)

1.  **Concept:** Securely generate a LiveKit JWT token on the server for the authenticated user to join *their* specific stream (room).
2.  **Add LiveKit Gem:**
    * `bundle add livekit-server-sdk`
3.  **Store Credentials Securely:**
    * `bin/rails credentials:edit` (This opens your default editor).
    * Add your LiveKit keys under a `livekit:` key:
        ```yaml
        livekit:
          api_key: YOUR_LK_API_KEY
          api_secret: YOUR_LK_API_SECRET
        ```
    * Save & close editor. This encrypts `config/credentials.yml.enc` using `config/master.key`. **Never commit `master.key`!**
4.  **Create Token Route:**
    * Open `config/routes.rb`. Inside the `resources :streams do ... end` block, add a `member` route:
        ```ruby
        resources :streams do
          member do
            post :generate_token # Use POST for actions that create/initiate something
          end
        end
        ```
    * Run `bin/rails routes | grep token` to verify the route exists (`POST /streams/:id/generate_token`).
5.  **Implement Token Action in Controller:**
    * Add this method to `app/controllers/streams_controller.rb`:
        ```ruby
        def generate_token
          # Find the stream scoped to the current user for security
          @stream = current_user.streams.find(params[:id])

          # Access credentials safely
          livekit_api_key = Rails.application.credentials.livekit[:api_key]
          livekit_api_secret = Rails.application.credentials.livekit[:api_secret]

          # Create LiveKit Access Token
          token = LiveKit::AccessToken.new(api_key: livekit_api_key, api_secret: livekit_api_secret)
          token.identity = current_user.id.to_s # Must be unique string identifier for the user
          token.name = current_user.email     # Optional: display name in LiveKit room
          token.add_grant(roomJoin: true,
                          room: @stream.id.to_s, # Use stream ID as room name (example)
                          canPublish: true,      # Adjust permissions as needed for your app
                          canSubscribe: true)

          # Return the token as JSON
          render json: { token: token.to_jwt }

        # Handle case where stream not found for this user
        rescue ActiveRecord::RecordNotFound
          render json: { error: "Stream not found or unauthorized" }, status: :not_found
        end
        ```
    * **Where to look:** LiveKit Server SDK (Ruby) documentation, Rails Guides: "Credentials".

**Goal:** Create a secure API endpoint that uses stored credentials and the LiveKit SDK to generate a JWT token specific to a user and stream.

---

## Hour 8: Basic Frontend & Next Steps (Approx. 45 mins)

1.  **Concept:** Use JavaScript on the stream's page (`show.html.erb`) to call the `generate_token` endpoint and get the token needed for the LiveKit Client SDK.
2.  **Minimal Frontend Integration:**
    * Edit `app/views/streams/show.html.erb`.
    * Add placeholders: `<div id="token-status">Fetching token...</div> <div id="livekit-video"></div>`
    * Add JavaScript (using a simple `<script>` tag for speed today):
        ```html
        <script src="[https://cdn.jsdelivr.net/npm/livekit-client/dist/livekit-client.umd.min.js](https://cdn.jsdelivr.net/npm/livekit-client/dist/livekit-client.umd.min.js)"></script>

        <script type="text/javascript">
          // Wait for the DOM to be fully loaded
          document.addEventListener('DOMContentLoaded', () => {
            const streamId = <%= @stream.id %>; // Get stream ID from Rails view variable
            const tokenUrl = `/streams/${streamId}/generate_token`;
            const tokenStatusDiv = document.getElementById('token-status');

            // Get the CSRF token from the meta tag Rails includes in the layout
            const csrfToken = document.querySelector("meta[name='csrf-token']").getAttribute("content");

            // Fetch the token from our Rails endpoint
            fetch(tokenUrl, {
              method: 'POST',
              headers: {
                'X-CSRF-Token': csrfToken, // Crucial for Rails POST requests from JS
                'Accept': 'application/json' // Indicate we want JSON back
              }
            })
            .then(response => {
              if (!response.ok) {
                // Throw an error to be caught by .catch()
                throw new Error(`HTTP error! Status: ${response.status}`);
              }
              return response.json(); // Parse the JSON body
            })
            .then(data => {
              const token = data.token;
              console.log("LiveKit Token:", token); // Log token to browser console
              tokenStatusDiv.innerText = "Token obtained successfully! Check console.";

              // ========================================
              // --- YOUR LIVEKIT CLIENT LOGIC HERE ---
              // You know LiveKit: Use the 'token' variable and your LiveKit WS URL
              // to connect the user to the room.
              // Example:
              // const room = new LiveKitClient.Room();
              // await room.connect('wss://YOUR_LIVEKIT_URL', token);
              // console.log('Connected to LiveKit room:', room.name);
              // Add logic to handle tracks, participants, UI updates...
              // ========================================

            })
            .catch(error => {
              console.error('Error fetching token or connecting to LiveKit:', error);
              tokenStatusDiv.innerText = `Error: ${error.message}`;
            });
          });
        </script>
        ```
    * **Important:** Ensure you have the `<%= csrf_meta_tags %>` helper in your `app/views/layouts/application.html.erb` (it's there by default). This provides the CSRF token.
3.  **Test:**
    * Restart `rails s`.
    * Log in and navigate to a stream's show page (e.g., `/streams/1`).
    * Open your browser's Developer Console (usually F12).
    * You should see the LiveKit token logged, and the "Token obtained..." message on the page.

**Goal:** Successfully fetch the server-generated LiveKit token from the frontend JavaScript, proving the backend communication works and is ready for your LiveKit client implementation.

---

## What's Next (Beyond Day 1)

* **Deepen Rails Understanding:** Revisit Rails Guides sections (Routing, Active Record, Action Controller, Action View Forms, Asset Pipeline/Importmaps/Webpacker).
* **Full LiveKit Client Implementation:** Build out the actual video/audio connection, UI, participant handling, etc., using the token.
* **Testing:** Learn Minitest (Rails default) or RSpec. Write tests for models, controllers, integration points. **This is critical.**
* **Refine Forms & Views:** Customize beyond scaffold defaults. Learn `form_with` options, partials.
* **Authorization:** Implement finer permissions (who can start/stop/join streams?). Look into gems like Pundit or CanCanCan.
* **Styling:** Integrate a CSS framework (TailwindCSS, Bootstrap) or write custom CSS.
* **Background Jobs:** For tasks like recording processing or notifications (Sidekiq, GoodJob).
* **Action Cable:** Explore Rails' built-in WebSocket solution for other real-time features if needed.
* **Deployment:** Learn to deploy (Heroku, Render, Fly.io, etc.).

---

Good luck with your Rails and LiveKit project! This one-day sprint gives you the essential foundation.
