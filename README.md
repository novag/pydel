# Pydel

A Python implementation of the Jodel protocol.

## Installation

After obtaining a copy of this repository, navigate to the directory and install it the same way you would install any
other Python package:

```
pip install -r requirements.txt
python setup.py install
```

I highly recommend installing to a [virtual environment](https://virtualenv.readthedocs.org/).

## Usage

Below is a brief overview of the methods available. Please refer to the documentation for more details, including which
exceptions might be raised.

### Authenticating

The Pydel class is used to communicate with the Jodel API. Its constructor takes 7 arguments:
 - city: The name of the city you are Jodling from
 - country code: The [country code](https://en.wikipedia.org/wiki/Country_code) for the country you are Jodling from
 - lat: Latitude of the location you're Jodling from
 - lng: Longitude of the location you're Jodling from
 - device_uid: A 64 character string consisting of numbers and lowercase numbers. The device UID is your "username", used to identify you.
 - (optional) user_agent_string: The user agent string you want to use. Default is "Jodel/65000 Dalvik/2.1.0 (Linux; U; Android 5.0; SM-G900F Build/LRX21T)"

```
from pydel import Pydel
import pydel.colors
from pydel.utils import random_device_uid

uid = random_device_uid()

p = Pydel(city='Trondheim', country_code='NO', lat=60.0, lng=10.0, device_uid=device_uid, user_agent_string='Jodel/65000 Dalvik/2.1.0 (Linux; U; Android 5.0; SM-G900F Build/LRX21T)')
p.authenticate()  # Authenticate with the server
```

Note that authenticate() must be called before doing anything else.

### Fetching data

get_karma() will return your karma as an integer value. Pydel also implements several public methods that you can use to
fetch posts: get_my_recent_posts(), get_my_popular_posts(), get_my_discussed_posts(), get_my_replies(), get_my_votes(),
get_recent_posts(), get_popular_posts() and get_discussed_posts(). These will all return a list of Post instances.

```
karma = p.get_karma()  # 42
top_jodels = p.get_popular_posts()  # [<pydel.Post instance at 0x7f798e7e9c20>, <pydel.Post instance at 0x7f798e7e9b00>, ...]
```

### Sending data
Pydel supports voting, replying and posting new jodels:

 - upvote_post/downvote_post(post) takes a Post instance and upvotes/downvotes it. Returns False if the user currently
  logged in has already voted on this post, True if the vote request was sent.
 - delete_post(post) attempts to delete the post associated with the given Post instance. Returns True if the request is
 sent without encountering any exceptions.
 - new_post(color, message) posts a new Jodel with the given color and message. Please note that the server will
 discard any non-Jodel colors.
 - new_reply(message, post) posts a reply containing message as a response to the given post.

```
p.new_post(color=pydel.colors.RED, message="I just love this app!")  # [<pydel.Post instance at 0x7f798e7e9c20>, <pydel.Post instance at 0x7f798e7e9b00>, ...]
```

### Colors
All colors are specified as a six character hexadecimal string. The options accepted by the server are
 - FF9908 (orange)
 - FFBA00 (yellow)
 - DD5F5F (red)
 - 06A3CB (blue)
 - 8ABDB0 (bluegreyish)
 - 9EC41C (green)

These can also be found as ORANGE, YELLOW, RED, BLUE, BLUEGREY and GREEN in pydel.colors:

```
print(pydel.colors.ORANGE)  # FF9908
```

### Post properties
A Post instance has methods upvote(), downvote() and reply(message). These behave just like the Pydel methods
described above. In addition, it has the following properties:
 - voted (str): "up"/"down" if the user fetching the post has voted on the post. None if the user has not voted.
 - vote_count (int): Signed integer indicating how many votes the post has.
 - has_replies (bool): True if the post has replies, False if it does not.
 - reply_from_op (bool): True if the post was made by someone replying to their own thread.
 - replies (list): List of Post objects representing the replies to this post. Empty list if there are no replies.
 - reply_count (int): The number of replies to this post.
 - is_image (bool): True if the post contains an image, False if it does not.
 - image_url (str): None if the post doesn't contain an image, AWS url if it does.
 - thumbnail_url (str): None if the post doesn't contain an image, AWS url if it does.
 - created_at (datetime): Time the post was created.
 - updated_at (datetime): Time the post was last updated (seems to always be the same as created_at).
 - own_post (boolean): True if the post was written by the user who fetched it, False if it was not.
 - distance (int): Euclidean post distance in kilometers (very_close: 0..1, close: 2..10, city_name: 11+).
 - location (dict): Dictionary mapping 'lat', 'lng' and 'name' to latitude, longitude and name.
 - message (str): The contents of the post. Empty string it no message is found.
 - color (str): Six character string describing the color of the post. FFFFFF if no color is found.
 - post_id (str): Alphanumeric string identifying the post.
 - user_handle (str): Alphanumeric string identifying a user in the current thread.
