# Implementation Notes: Adding front_teammate_id to AdminUser Model

## Context
Based on the Linear story GCM-3080, we need to include `authorId` in messages sent via Front API. This requires:

1. Adding `front_teammate_id` attribute to the AdminUser model
2. Including this attribute in the `send_reply` function call

## Current Situation
This workspace appears to contain documentation files (Mintlify) rather than the application source code. The AdminUser model and send_reply function are not present in the current workspace.

## Implementation Steps Required

### 1. AdminUser Model Changes

**For Django (Python):**
```python
# In your AdminUser model (likely in models.py)
class AdminUser(models.Model):
    # ... existing fields ...
    front_teammate_id = models.CharField(max_length=255, null=True, blank=True)
    
    # ... rest of the model ...
```

**For Rails (Ruby):**
```ruby
# In your AdminUser model (likely in app/models/admin_user.rb)
class AdminUser < ApplicationRecord
  # ... existing code ...
  
  # Add this line if using a migration:
  # rails generate migration AddFrontTeammateIdToAdminUsers front_teammate_id:string
end
```

### 2. Database Migration

**Django:**
```bash
python manage.py makemigrations
python manage.py migrate
```

**Rails:**
```bash
rails generate migration AddFrontTeammateIdToAdminUsers front_teammate_id:string
rails db:migrate
```

### 3. Update send_reply Function Call

Find where `send_reply` is called and modify it to include the `front_teammate_id`:

**Example (Python):**
```python
# Before
send_reply(message_content, recipient, ...)

# After
send_reply(
    message_content, 
    recipient, 
    author_id=admin_user.front_teammate_id,
    ...
)
```

**Example (Ruby):**
```ruby
# Before
send_reply(message_content, recipient, ...)

# After
send_reply(
  message_content, 
  recipient, 
  author_id: admin_user.front_teammate_id,
  ...
)
```

### 4. Update send_reply Function Definition

The `send_reply` function itself may need to be updated to accept and use the `author_id` parameter:

**Example (Python):**
```python
def send_reply(message_content, recipient, author_id=None, **kwargs):
    payload = {
        'content': message_content,
        'recipient': recipient,
        # ... other fields ...
    }
    
    if author_id:
        payload['author_id'] = author_id
    
    # Send to Front API
    # ...
```

## Next Steps

1. Navigate to the actual application codebase
2. Locate the AdminUser model file
3. Add the `front_teammate_id` field
4. Create and run the database migration
5. Find all calls to `send_reply`
6. Update the function calls to include the `front_teammate_id`
7. Update the `send_reply` function definition if needed
8. Test the integration with Front API

## Files to Look For

- **Django:** `models.py`, `views.py`, or service files containing AdminUser and send_reply
- **Rails:** `app/models/admin_user.rb`, controller files, or service objects
- **API Integration:** Look for Front API integration code, possibly in a separate service or utility file

## Testing

1. Verify the new field is added to the database
2. Test that `send_reply` correctly includes the `author_id` when an AdminUser has a `front_teammate_id`
3. Verify the Front API receives the correct `author_id` in the message payload