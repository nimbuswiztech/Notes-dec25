# Behind the scenes: From the moment you enter a URL

The overall picture

Although, a webpage loads in a matter of seconds, there is quite a lot going on in the background. For simplicity, we will split them into three major flows:

**1. Get the IP address of the server that your domain name refers to**

[![Alt Text](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fi%2F0x0qzu9xi1av6ct2dnmm.png)](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fi%2F0x0qzu9xi1av6ct2dnmm.png)

**2. Hit the server to fetch what is to be rendered on the UI**

[![Alt Text](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fi%2Fpx0cu6wb80fh107gv40p.png)](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fi%2Fpx0cu6wb80fh107gv40p.png)

**3. Construct, paint, and render the page**

[![Alt Text](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fi%2Fs25rhzjnd32gt9rpk968.png)](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fi%2Fs25rhzjnd32gt9rpk968.png)

This is it. It's as simple as three steps. However, the complex part kicks in when we dive deeper into the two black boxes-

1. IP address resolution.
2. Constructing and rendering webpage.

If you are backend developer, the first will be of prime concern to you, and for frontend folks it's the browser rendering that takes precedence.\
Anyways, let's look into both of them.

### Domain name to IP Address using Domain Name Server(DNS) <a href="#domain-name-to-ip-address-using-domain-name-serverdns" id="domain-name-to-ip-address-using-domain-name-serverdns"></a>

Although this article is not meant for heavy theory (there are plenty on the internet), I will give a small summary why this block is important.

We as human beings don't retain long numbers. And machines don't understand our sophisticated language. As a win-win solution, we give **names** (domain name) to our servers while they have their identity as IP Addresses (numbers). So how do we bridge the gap and communicate? **Domain Name Server** acts as our mediator!\
This is what goes behind the curtains:

[![Alt Text](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fi%2Fnmgeburj0q5o1vazjkxr.png)](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fi%2Fnmgeburj0q5o1vazjkxr.png)\
&#xNAN;_&#x53;orry if image is too small on your device, I had to fit in a lot of content in one flow chart. I urge you to download it and analyse each component. They are quite self-explanatory._

### How the user finally gets a fancy webpage <a href="#how-the-user-finally-gets-a-fancy-webpage" id="how-the-user-finally-gets-a-fancy-webpage"></a>

Once we have resolved the IP Address of the server that has our relevant data (the webpage), all there is left is to actually hit it and fetch what we wanted. Most of the time, we get an HTML page in response, but we also have instances when it is a PDF, or other content-types like image, JSON, XML etc.

In this section, we will see how the browser converts an HTML file (bunch of nodes, scripts, and stylings) into a full-fledged viewable page.

[![Alt Text](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fi%2Fyn1m5ni71ykq992o6z2b.png)](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fi%2Fyn1m5ni71ykq992o6z2b.png)

This is only an overview. But, if you want to dig deeper into how each and every browser component works, refer [here](https://www.html5rocks.com/en/tutorials/internals/howbrowserswork/#Introduction). This site is pretty much the Magnum Opus of how browser renders the HTML with embedded scripts and painted stylings.

However, I would like to mention a couple of important points here-\
1- Your **Browser Engine** holds your JS environment like v8(for chrome) that has call stack, memory heap, event loop, Web API.. yada yada.\
2- It's the **Render Engine** that parses the HTML nodes into a DOM tree and then further into a painted(CSS applied) render tree to display.\
3- Everytime your HTML parser encounters **script** tag, it **PAUSES PARSING DOM elements** (IMPORTANT!!!) and synchronously downloads all scripts first.

### Conclusion <a href="#conclusion" id="conclusion"></a>

Again, the agenda of this article was to help you articulate a big picture into a consolidated 3 min answer if anybody ever asks you- _"What happens when you enter a URL?"_. Of course, there is a lot to explore here, and there are brilliant sources online to do so. Mentioning some of them in the references below.
