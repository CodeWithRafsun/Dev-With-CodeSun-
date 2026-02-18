# Supabase Database Setup Guide

## Why the App Was Crashing

The app requires three database tables to function:
- `profiles` - Store user information
- `api_keys` - Store encrypted Gemini API keys
- `history` - Store user interaction history

Without these tables, the app crashes on startup when it tries to access the database.

## Quick Setup (5 minutes)

### 1. Open Supabase SQL Editor
- Go to https://supabase.com and log in
- Select your CodeSun project
- Click **"SQL Editor"** in the left menu

### 2. Create Tables
Copy and paste this entire SQL block into the editor and click **"Run"**:

```sql
-- Create profiles table
CREATE TABLE profiles (
  id UUID REFERENCES auth.users(id) PRIMARY KEY,
  name TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Create api_keys table
CREATE TABLE api_keys (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES auth.users(id) UNIQUE NOT NULL,
  gemini_key TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Create history table
CREATE TABLE history (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES auth.users(id) NOT NULL,
  tool_type TEXT NOT NULL,
  input TEXT,
  output TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Enable Row Level Security
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE api_keys ENABLE ROW LEVEL SECURITY;
ALTER TABLE history ENABLE ROW LEVEL SECURITY;

-- Create policies for profiles
CREATE POLICY "Users can view their own profile" ON profiles
  FOR SELECT USING (auth.uid() = id);

CREATE POLICY "Users can update their own profile" ON profiles
  FOR UPDATE USING (auth.uid() = id);

-- Create policies for api_keys
CREATE POLICY "Users can view their own API key" ON api_keys
  FOR SELECT USING (auth.uid() = user_id);

CREATE POLICY "Users can insert their own API key" ON api_keys
  FOR INSERT WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can update their own API key" ON api_keys
  FOR UPDATE USING (auth.uid() = user_id);

CREATE POLICY "Users can delete their own API key" ON api_keys
  FOR DELETE USING (auth.uid() = user_id);

-- Create policies for history
CREATE POLICY "Users can view their own history" ON history
  FOR SELECT USING (auth.uid() = user_id);

CREATE POLICY "Users can insert their own history" ON history
  FOR INSERT WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can delete their own history" ON history
  FOR DELETE USING (auth.uid() = user_id);
```

### 3. Verify Tables Created
- Click **"Table Editor"** in the left menu
- You should see three tables: `profiles`, `api_keys`, `history`

### 4. Reinstall App
- Uninstall the app from your phone
- Reinstall the APK
- The app should now work without crashing!

## What Each Table Does

| Table | Purpose |
|-------|---------|
| `profiles` | Stores user name and account info |
| `api_keys` | Stores encrypted Gemini API keys (one per user) |
| `history` | Stores all user interactions with the AI tools |

## Row Level Security (RLS)

The SQL commands above enable RLS, which ensures:
- Users can only see their own data
- Users cannot access other users' API keys or history
- All data is protected at the database level

## Troubleshooting

**Error: "relation 'profiles' already exists"**
- The tables already exist, you can skip this step

**Error: "permission denied"**
- Make sure you're using a service role key with full permissions
- Or use the Supabase dashboard SQL editor (no authentication needed)

**App still crashes after setup**
- Clear app cache: Settings → Apps → CodeSun → Storage → Clear Cache
- Uninstall and reinstall the app
- Check that SUPABASE_URL and SUPABASE_ANON_KEY are correct in your .env file

## Next Steps

After database setup:
1. Open the app and create an account
2. Go to Settings and add your Gemini API key
3. Try the Prompt Generator tool
4. Enjoy using CodeSun!

---

Need help? Check the main README.md or IMPLEMENTATION_GUIDE.md for more details.
