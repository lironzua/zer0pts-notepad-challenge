# zer0pts-notepad-challenge
A solution for the zer0pts ctf notepad challenge

At first sight we see a flask web application.

The app manages notes that created by a user. Every user can create, edit and delete his notes, which is saved in the session flask variable which is saved encrypted in the user cookie. 

The key of the encryption is a part of the flask application config and is defined in line `9`.

You can look at the code and see that the data is saved under the session member `savedata`, which is an array of objects that represents notes.

The application uses Jinja2 as the template library as it’s the default of the framework. 

The application is kinda stateless so we knew it must be some kind of command injection, which is common in flask.

Looking at the 404 handler we see some place where we can inject a template string, which will be interpolated at server side. The handler checks whether the referrer is from the same host and that it does not exceed 16 characters after the host name itself, so we are limited in the string we can enter.

![Image description](https://i.imgur.com/NxsyZmN.png)

So we constructed a new http request and placed {{config}} after the host in the referrer:

![Image description](https://i.imgur.com/FZuKqHK.png)

And we successfully extracted the app’s secret_key 😊
What can we do with it? We can decrypt the data from the cookie which we already have access to but that doesn’t make sense. According to the code it only contains the note array, but, we can change the data and encrypt it again using the key.
Using this code I decrypted the key, with some manipulation you’re also able to encrypt it again - https://gist.github.com/babldev/502364a3f7c9bafaa6db

As we can see, the app uses pickle for serialization which is dangerous, as pickle known to be vulnerable to attacks.
We can serialize an object like this:

![Image description](https://i.imgur.com/2JrLcVd.png)

The __reduce__ function will be triggered by pickle.loads which will run the command in the string.


You can read more about pwning pickle here: https://root4loot.com/post/exploiting_cpickle/
You can read more about flask injections here: https://nvisium.com/blog/2015/12/07/injecting-flask.html

