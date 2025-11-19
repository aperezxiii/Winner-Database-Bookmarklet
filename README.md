# Winner-Database-Bookmarklet
Used to create a bookmarklet so anyone can make a pull.
## Bookmarklet

To use the Winner Database console as a bookmarklet, create a new browser bookmark and set its **URL/location** to this exact code:

```javascript
javascript:(function(){var s=document.createElement('script');s.src="https://cdn.jsdelivr.net/gh/aperezxiii/Winner-Database-Bookmarklet@main/console.js";document.body.appendChild(s);})();
