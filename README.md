**Spotify Clone: A Scalable Music Streaming Application**

This project is a full-stack, dynamic music streaming application built as a modern web development exercise. It aims to replicate core functionalities of a service like Spotify, including user authentication, song playback, and playlist management, with a strong focus on data security and real-time updates.

Technologies Used
Frontend:

React: A JavaScript library for building user interfaces.

Next.js: A React framework that enables server-side rendering and static site generation, ensuring fast performance.

Tailwind CSS: A utility-first CSS framework for rapid and responsive styling.

Backend & Database:

Supabase: An open-source Firebase alternative providing a complete backend solution. It offers a PostgreSQL database, authentication, and a real-time engine.

Hosting:

Vercel: A platform for building and deploying Next.js applications with ease.

Key Features
User Authentication: Secure sign-up and log-in functionality.

Dynamic UI: The user interface updates in real-time as data changes.

Song Management: Users can upload, view, and play songs.

Playlist Creation: Users can create and manage their own playlists.

How It Works
The application follows a modern server-side rendered architecture. The Next.js frontend handles the user interface and data fetching. When a user interacts with the app, such as uploading a new song, a request is sent to the Supabase API. Supabase, in turn, handles the database operations and uses its real-time capabilities to push updates to all relevant users, ensuring a seamless and dynamic experience.

Security & Data Protection with Supabase
A primary focus of this project is data integrity and user security, which is managed through Supabase's Row-Level Security (RLS) policies. RLS is a powerful database feature that allows us to define fine-grained rules directly on our database tables, restricting what data a user can access, insert, update, or delete.

The core of our security model is a policy that ensures a user can only interact with their own data.

The Policy: On tables containing private user data (e.g., playlists or songs), we have implemented a policy that states:

"A user is only allowed to READ or WRITE to a row if the user_id column in that row matches the id of the currently authenticated user."

How it is enforced: This policy is written in SQL and is enforced by the database itself, not by our application code. This provides an extra layer of security, as any attempt to bypass the application's logic and directly query the database will be blocked by the RLS policy. For example, a user cannot accidentally or maliciously view another user's private playlist because the database itself will prevent the query from returning that data.

Getting Started
To get a copy of this project up and running locally, follow these steps.

Clone the repository:

git clone https://github.com/AyeshaNahman/Spotify-Clone.git
cd Spotify-Clone

Set up your Supabase project:

Create a new project on Supabase.

Navigate to the SQL Editor and run the following queries to set up your tables and RLS policies. These tables are inspired by a Stripe-like payments template and additional tables for music functionality.

Stripe-Inspired Tables:

-- Create the 'users' table
create table public.users (
  id uuid references auth.users not null primary key,
  full_name text,
  avatar_url text,
  billing_address jsonb,
  payment_method jsonb
);

-- Secure the 'users' table with RLS
alter table public.users enable row level security;
create policy "Can view own user data." on public.users for select using (auth.uid() = id);
create policy "Can update own user data." on public.users for update using (auth.uid() = id);

-- Create the 'products' table for subscription tiers
create table public.products (
  id text primary key,
  active boolean,
  name text,
  description text,
  image text,
  metadata jsonb
);

-- Secure the 'products' table with RLS
alter table public.products enable row level security;
create policy "Products are viewable by everyone." on public.products for select using (true);

-- Create the 'prices' table for product pricing
create table public.prices (
  id text primary key,
  product_id text references public.products(id),
  active boolean,
  unit_amount bigint,
  currency text check (char_length(currency) = 3),
  type text,
  billing_scheme text,
  interval text,
  interval_count integer,
  trial_period_days integer
);

-- Secure the 'prices' table with RLS
alter table public.prices enable row level security;
create policy "Prices are viewable by everyone." on public.prices for select using (true);

-- Create the 'subscriptions' table
create table public.subscriptions (
  id text primary key,
  user_id uuid references public.users(id),
  status text,
  metadata jsonb,
  price_id text references public.prices(id),
  quantity integer,
  cancel_at_period_end boolean,
  created_at timestamp with time zone,
  ended_at timestamp with time zone,
  cancel_at timestamp with time zone,
  canceled_at timestamp with time zone
);

-- Secure the 'subscriptions' table with RLS
alter table public.subscriptions enable row level security;
create policy "Can only view own subscriptions." on public.subscriptions for select using (auth.uid() = user_id);

Additional Music-Specific Tables:

-- Create the 'songs' table
create table public.songs (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references public.users(id) not null,
  title text,
  author text,
  song_path text not null,
  image_path text
);

-- Secure the 'songs' table with RLS
alter table public.songs enable row level security;
create policy "Songs are viewable by everyone." on public.songs for select using (true);
create policy "Users can insert their own songs." on public.songs for insert with check (auth.uid() = user_id);
create policy "Users can delete their own songs." on public.songs for delete using (auth.uid() = user_id);

-- Create the 'playlists' table
create table public.playlists (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references public.users(id) not null,
  title text,
  is_public boolean default false
);

-- Secure the 'playlists' table with RLS
alter table public.playlists enable row level security;
create policy "Playlists can be viewed if public or owned." on public.playlists for select using (is_public = true OR auth.uid() = user_id);
create policy "Users can insert their own playlists." on public.playlists for insert with check (auth.uid() = user_id);
create policy "Users can update their own playlists." on public.playlists for update using (auth.uid() = user_id);
create policy "Users can delete their own playlists." on public.playlists for delete using (auth.uid() = user_id);

-- Create a 'playlist_songs' join table
create table public.playlist_songs (
  playlist_id uuid references public.playlists(id),
  song_id uuid references public.songs(id),
  primary key (playlist_id, song_id)
);

-- Secure the 'playlist_songs' join table with RLS
alter table public.playlist_songs enable row level security;
create policy "Users can add songs to their own playlists." on public.playlist_songs for insert with check ((select user_id from public.playlists where id = playlist_id) = auth.uid());
create policy "Users can view songs in a public playlist." on public.playlist_songs for select using ((select is_public from public.playlists where id = playlist_id) = true);
create policy "Users can view songs in their own playlist." on public.playlist_songs for select using ((select user_id from public.playlists where id = playlist_id) = auth.uid());
create policy "Users can remove songs from their own playlists." on public.playlist_songs for delete using ((select user_id from public.playlists where id = playlist_id) = auth.uid());

After running the queries, copy your Supabase project URL and public key from your project's settings.

Configure environment variables:

Create a .env.local file in the root of your project.

Add your Supabase credentials:

NEXT_PUBLIC_SUPABASE_URL=YOUR_SUPABASE_URL
NEXT_PUBLIC_SUPABASE_ANON_KEY=YOUR_SUPABASE_ANON_KEY

Install dependencies and run the development server:

npm install
npm run dev

Open http://localhost:3000 in your browser to see the application.

🙏 Acknowledgements
Supabase for the excellent open-source backend solution.

Next.js for the powerful React framework.

Tailwind CSS for simplifying the styling process.
