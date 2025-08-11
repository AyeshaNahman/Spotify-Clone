# Spotify Clone 🎧

A beautiful, modern music streaming application built with Next.js and Supabase.

[![Next.js](https://img.shields.io/badge/Next.js-13.4.19-black)](https://nextjs.org/)
[![React](https://img.shields.io/badge/React-18.2.0-blue)](https://reactjs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.1.6-3178C6)](https://www.typescriptlang.org/)
[![Supabase](https://img.shields.io/badge/Supabase-4.2.3-3ECF8E)](https://supabase.io/)
[![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-3.3.3-38B2AC)](https://tailwindcss.com/)

---

##  Features

###  Song Management

* **Upload**: Securely upload songs and associated images.
* **Play**: Seamlessly stream music with a responsive player.
* **Discovery**: Browse and play songs uploaded by other users.

###  User & Playlist
* **Authentication**: Secure sign-up and log-in functionality powered by Supabase Auth.
* **Dynamic Playlists**: Create and manage personal playlists.
* **Real-time Updates**: The UI updates automatically with new data.

---

##  Technologies

| Category | Technologies |
| :--- | :--- |
| **Frontend** | Next.js, React, TypeScript |
| **Styling** | Tailwind CSS |
| **Database** | Supabase (PostgreSQL) |
| **Charts** | Recharts |
| **Authentication** | Supabase Auth |


---

##  How to Get Started

### Prerequisites

You'll need `Node.js 15+` and `npm 7.0.0+` installed on your machine.

```
node --version # Should be 15.0.0 or higher
npm --version  # Should be 7.0.0 or higher
```

### Installation

1.  **Clone the repository**:

    ```
    git clone [https://github.com/AyeshaNahman/Spotify-Clone.git](https://github.com/AyeshaNahman/Spotify-Clone.git)
    cd Spotify-Clone
    ```

2.  **Set up your Supabase project**:

    * Create a new project on Supabase.
    * Navigate to the **SQL Editor** and run the following queries to set up your tables and Row-Level Security (RLS) policies.

    ```
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
    -- Additional Music-Specific Tables
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
    ```

3.  **Configure environment variables**:

    * Copy your Supabase project URL and public key from your project's settings.
    * Create a `.env.local` file in the root of your project.
    * Add your Supabase credentials to the file:

    ```
    NEXT_PUBLIC_SUPABASE_URL=YOUR_SUPABASE_URL
    NEXT_PUBLIC_SUPABASE_ANON_KEY=YOUR_SUPABASE_ANON_KEY
    ```

4.  **Install dependencies and run the development server**:

    ```
    npm install
    npm run dev
    ```

    Open `http://localhost:3000` in your browser.

---

##  Acknowledgements

* **Supabase** for the powerful open-source backend solution.
* **Next.js** for the robust React framework.
* **Tailwind CSS** for simplifying the styling process.
