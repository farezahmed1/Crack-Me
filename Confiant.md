# CrackMe Report
For this challenge, I was given the task of finding Captain Platinumbeards' “booty.” In order to do this, I needed to employ many built-in tools native to my browser to view the JavaScript (inspecting the page source) and view the interactions of such a script (using the console). 
This allowed me to decipher the conditions necessary to uncover the treasure and solve the puzzle. Subsequently, I documented the solution, showcasing my approach and little tweaks I had to do along the way to make my life easier when reverse engineering the script. 

## Solution
There were many requirements that needed to be met in order for the solution to come to light. This hunt required me to get all of the “runes” within the page by moving the interactable X marker. 
However, one may only find the treasure if moving the X marker while a certain condition is met - and that condition was to have the correct “clue” be active (the correct sailors name I assumed). 
In order to do that, I needed to change the browser’s `userAgent` to read `BLACKSEA` starting at the 9th index. Here is an example shown in Google Chrome:

![](https://i.imgur.com/G5bYhwQ.png)

### Gathering the Runes
After setting the correct `userAgent`, I had to move the marker to three specific locations to acquire these `runes`. These `runes` have corresponding names to them once you reach them and they are in this order: 
`alpha`, `beta`, `gamma`. Here are examples of what occurs when you reach these runes in the correct order from a console point of view (I did tweak the source code in order to provide some sanity checks while the process was happening):

`alpha` was located at (73, 82)

![](https://i.imgur.com/l2EDQzq.png)

`beta` was located at (76, 80)

![](https://i.imgur.com/WkeFKuM.png)

`gamma` was located at (89, 69)

![](https://i.imgur.com/AeeNBZt.png)

## Methodology
The only “tools” I used throughout this challenge is Notepad++ to change the source code for sanity checks. I did have to conduct some research on some functions that I was not used to using as often. Everything else used was built into the Chrome browser’s dev tools panel.

### Step 1: Inspection
The very first thing I did was read the prompt to see if there were any hidden messages within it. After deciding there probably is not, I opened up the source code via the inspect tool found in Chrome’s browser. 
I was immediately hit with a debugger cue that looked to be oddly stuck. I assumed this was all a part of the plan from stopping me from finding the treasure. I scrolled throughout the entirety of the script and noticed there were quite a few functions that were being defined, 
as well as built in functions that were being used. I slowly took a mental note of all of the variables and functions that were being defined and then shortly after, closed the inspect panel in Chrome.

At this point I decided to tinker with the actual page to see how I would go about finding this treasure. I immediately noticed the X marker and started to click around it until it moved. I now knew I could drag the X marker to certain spots on the page, like a pirate navigating maps. 

### Step 2: Decoding
Early on in my time inspecting the script, I noticed a lot of mumbo jumbo being assigned to variables such as spikes and ee. I saw that `b = atob` which made me think that atob could be a builtin function of some sort. Upon doing a quick Google search, 
I learned it was the function that decodes a string of data which has been encoded using Base64 encoding. I then tried to use the `atob` function assigned to the variable `b` on as many things as I could that was applicable. I decoded all of the strings in the spikes array, 
the item that was in the variable `ee`, and what looked to be the win alert on line 75. 
Here is a list of the decoded items:

Line 36 - `spikes = ['navigator', 'userAgent', 'indexOf', 'LAMBERT', 'RIPLEY', 'ALDERSON', 'BLACKSEA', 'dir', 'clear', 'clue', 'canary', 'alpha', 'beta', 'gamma']`

Line 75 - `eval('alert("Congratulations Oh 1337 One! You have found my treasure!")` 

Upon trying to decode everything, I noticed that some of the items that were wrapped within the function ‘b’, had a nested ‘tY’ function. I went to decipher what the tY function was trying to do, and ultimately figured out that it just swapped the first and the last character of the 
parameter/string. With this new information, I was able to decipher some more parts of the code:

- Line 47 - `window['navigator']['userAgent'].indexOf('BLACKSEA') == 9`

- Line 48 - `‘RIPLEY’`

- Line 50 - `‘LAMBERT’`

- Line 57 - `‘ALDERSON’`

I didn’t know what they meant just yet, but things were about to get interesting.

### Step 3 - Win conditions
Upon looking further down the code, I read the if statement above the win alert and it looks like in order for the alert to trigger, it must satisfy two conditions. The first condition is that the `runes` array must harbor more than 2 elements. 
The second condition is that the value stored in the `clue` variable satisfies the nameless function with the parameter named `targ`. These two conditions are what would allow the win alert to show based on what was previously decoded.

The only other function that appends anything to the `runes` array was the `IChing` function that took the x and y coordinates as the parameters. Upon reading the function, it was clear that the runes alpha, beta and gamma would be appended (using the `.push` method) 
once the X marker landed on the correct spots on the page. However, these spots must be triggered in a specific order - coincidentally, in the order of the compound conditional statements on lines 87, 89 and 91. This was deduced by the `runes.length` condition 
within each compound conditional statement, suggesting that a specific `runes` length must be met in order to add an element into `runes` - satisfying this condition for the next compound conditional statement. 

Within each compound conditional statement the y and x variables need to equate to the ASCII representation of the different characters within the string currently assigned to the variable `clue`. With that being said, it is critical 
that we have the right value assigned to the variable `clue`. According to the initialization of the `clue` variable, `clue` can have one of two values, `‘RIPLEY’` and `‘LAMBERT’`. Based on what was previously deduced, `‘RIPLEY’` would need to be assigned to the 
variable `clue` to have the correct x, y coordinates to find the treasure. If `‘LAMBERT’` were to be assigned to the variable `clue` while being passed through the `IChing` function, not only will it provide incorrect coordinates but it would also dissatisfy 
the second of the two conditions that was previously stated.

> [!IMPORTANT]
> We are making sure to have the correct value assigned to the variable `clue` such that it reveals the correct coordinates to navigate to with the X marker.

### Step 4: Finding the right clue
After decoding the entirety of the `clue` variable using the atob function, I had a clearer picture on how the function reads. I made an educated guess and inferred that the variable `clue` must harbor the first return statement of the if statement, since the else 
statement would be the constant default otherwise. This was confirmed by viewing the side panel of the dev tools in Chrome.

![](https://i.imgur.com/BfGrQex.png)

Now the goal was to identify the condition that makes the variable `clue` have `‘RIPLEY’` assigned to it since that was the first return statement of the if statement. I checked the condition required for this to happen (the first line of the function) 
and it read like this: `window['navigator']['userAgent'].indexOf('BLACKSEA') == 9`. 

Upon doing some Google research, I found that the window object is home to a variety of functions, namespaces, objects and constructors which are not necessarily directly associated with the concept of a user interface window. 
This meant it may be browser related and not UI related. I followed the trail and looked up what `navigator` and `userAgent` could mean and voila, I learned that the `navigator` is an attribute of the window object and `userAgent` is an attribute of the navigator object. I learned that you can change the user agent 
within the Chrome browser to something customized. I then changed the browser's user agent to start reading `'BLACKSEA'` at the 9th index as the line of code suggests in order for the variable `clue` to have assigned `‘RIPLEY’` to it. After doing so, the variable `clue` now had `‘RIPLEY’` assigned to it. 

![](https://i.imgur.com/pVhzJoa.png)

Now that we have the correct clue in place, we could proceed to find the treasure. 

### Step 5: Putting it all together
It was now time to sail the seas. I had one problem however - I can’t sail the seas with this pesky debugger prompt preventing me from moving the marker in real-time. It was crucial I had access to the console to view the coordinates in real-time while I was moving the marker. At this point, I had downloaded the HTML 
file and made a couple of tweaks to provide some quality of life using Notepad++. Here are some of the changes I had made:

- I had removed `ee('dLb'.replace('L', 'e') +'ug' + '14r'.replace('14', 'ge'))` from the code in order for me to move the marker in real-time while viewing the console.

- I had removed `console.dir(el)` since I did not need to view the properties at this time.

- I had removed `console.clear()` from the code in order to prevent it from clearing the entries in the console.

The latter two items I had removed were also revealed in the `spikes` array. This may have indicated that I needed to tinker with them in the future.

One of the final issues that I needed to solve was the issue of seeing the coordinates the marker was on in real-time. Nothing in the source code allowed the logging of such information. I looked at the div tag with the id of `‘gps’` and realized that the source code used `‘gps’` 
to display the encoded representation of the current coordinates on the screen: `document.getElementById("gps").innerHTML = btoa(y.toString() + "," + x.toString()).replace("=", "\n\<br>\\n")`. However, this representation isn’t useful.

Alongside the inner HTML provided by the initial source code, I also displayed the x and y coordinates in the user interface such that I know where my X marker is positioned. Now that we have a clear way to identify the coordinates the marker is residing at, we have to match it with the ASCII 
representation of the characters of the variable assigned to `clue` (`‘RIPLEY’`) using the `.charCodeAt` method and an ASCII table. At this point, I had the coordinates that I needed to traverse sequentially (based on the `IChing` function’s compound conditions statements) in order to reach the final 
destination, also referred to as fulfilling the win condition stated in step 3. 

> [!NOTE]
> The coordinates in the compound conditional statements in the `IChing` function were shown in the format of y first and x second. This is important as this might have thrown me off if I did not catch it since the traditional way of noting coordinates are x first and y second. 

These were the final deduced coordinates:

ASCII (I) = 73 ASCII (R) = 82 | First rune location (73, 82)
ASCII (L) = 76 ASCII (P) = 80 | Second rune location (76,80)
ASCII (Y) = 89 ASCII (E) = 69 | Third rune location (89,69)

Upon moving to these fixed locations using my real-time coordinate line of code that I added to help me, I could see the `runes` array population (another piece of code that I added to show this in real-time). Once I reached the final position, the win alert popped up and that indicated that I had found the treasure!


