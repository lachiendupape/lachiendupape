from atproto import Client
import praw
import schedule
import time
from datetime import datetime

# Authenticate with Bluesky
client = Client()
client.login('reddittest.bsky.social', '8084F3tt!!')

#get did
from atproto import IdResolver  # for async use AsyncIdResolver

resolver = IdResolver()
did = resolver.handle.resolve('reddittest.bsky.social')
did_doc = resolver.did.resolve(did)

print(did)
print(did_doc)

# Set up Reddit client
reddit = praw.Reddit(
    client_id="r4Wg_3jOfcWQYmDJff3NWg",
    client_secret="bVSX5k5AzcmAi6U9zaNL5OqWdqFWTA",
    user_agent='bsky bot'
)

# File to store IDs of posts that have been posted to Bluesky
POSTED_POSTS_FILE = 'posted_posts.txt'

# Function to load posted post IDs from file
def load_posted_ids():
    try:
        with open(POSTED_POSTS_FILE, 'r') as file:
            return set(line.strip() for line in file)
    except FileNotFoundError:
        return set()

# Function to save a new post ID to the file
def save_posted_id(post_id):
    with open(POSTED_POSTS_FILE, 'a') as file:
        file.write(f"{post_id}\n")

# function to clear post IDs
def clear_posted_ids():
    with open(POSTED_POSTS_FILE, 'w') as file:
        file.write("")  # Overwrites the file with an empty string
    print(f"Cleared all saved post IDs from {POSTED_POSTS_FILE}")

# Function to post to Bluesky using rich text
def post_to_bluesky(title, url, subreddit_url, thumbnail_url=None):
    text = f"{title}\n\nRead more here.\n\n#bhafc #reddit\n\nCome join the conversation!"

    facets = [
        {
            "index": {"byteStart": 0, "byteEnd": len(title)},
            "features": [
                {
                    "$type": "app.bsky.richtext.facet#link",
                    "uri": url
                }
            ]
        },
        {
            "index": {"byteStart": text.find("Read more here"), "byteEnd": text.find("Read more here") + len("Read more here")},
            "features": [
                {
                    "$type": "app.bsky.richtext.facet#link",
                    "uri": url
                }
            ]
        },
        {
            "index": {"byteStart": text.find("Come join the conversation"), "byteEnd": text.find("Come join the conversation") + len("Come join the conversation")},
            "features": [
                {
                    "$type": "app.bsky.richtext.facet#link",
                    "uri": subreddit_url
                }
            ]
        }
    ]
 
 # Add thumbnail link to facets if available
    if thumbnail_url:
        facets.append({
            "index": {"byteStart": text.find("Thumbnail:"), "byteEnd": text.find("Thumbnail:") + len("Thumbnail:")},
            "features": [
                {
                    "$type": "app.bsky.richtext.facet#link",
                    "uri": thumbnail_url
                }
            ]
        })

 # Create the post data
    post_data = {
        "repo": did,
        "collection": "app.bsky.feed.post",
        "record": {
            "$type": "app.bsky.feed.post",
            "text": text,
            "facets": facets,
            "createdAt": datetime.utcnow().isoformat() + "Z"
        }
    }
    
# Post the record using the AT Protocol client
    client.com.atproto.repo.create_record(post_data)
    print(f"Posted to Bluesky: {text}")

# Clears the file when called
clear_posted_ids() 

# Function to fetch top posts and post them to Bluesky
def fetch_top_posts():
    subreddit_name = 'BrightonHoveAlbion'
    subreddit_url = f"https://www.reddit.com/r/{subreddit_name}"
    posted_ids = load_posted_ids()

    print(f"Fetching top posts from r/{subreddit_name} at {datetime.now()}")

    # Fetch top posts from the past day
    subreddit = reddit.subreddit(subreddit_name)
    for post in subreddit.top(time_filter='day', limit=5):
        if post.id in posted_ids:
            print(f"Skipping already posted post: {post.title}")
            continue

        # Post to Bluesky
        post_to_bluesky(post.title, post.url, subreddit_url)

        # Save the post ID to the file
        save_posted_id(post.id)

# Schedule the bot to run twice daily
schedule.every().day.at("09:00").do(fetch_top_posts)  # Morning
schedule.every().day.at("18:22").do(fetch_top_posts)  # Evening

print("Bot started. Fetching top posts twice daily...")

# Run the scheduled jobs
while True:
    schedule.run_pending()
    time.sleep(1)