# CrackMe Report
In tackling this challenge, I undertook the mission of locating Captain Platinumbeard's "booty." To accomplish this, I utilized various built-in browser tools to inspect the JavaScript (examining the page source) and observe the script's interactions (using the console). 
This approach enabled me to decipher the conditions necessary to unveil the treasure and solve the puzzle. Following this, I meticulously documented the solution, outlining my methodology and the adjustments made during the reverse engineering process.

## Solution
The solution required meeting specific criteria to reveal itself. I needed to collect all the "runes" on the page by manipulating the interactive X marker. However, the treasure would only be accessible if the X marker was moved under a specific condition - matching the correct "clue".
To achieve this, I had to modify the browser's userAgent to start reading as BLACKSEA from the 9th index. An example in Google Chrome is illustrated below:

![](https://i.imgur.com/G5bYhwQ.png)

### Gathering the Runes
After setting the correct userAgent, I navigated the marker to three specific locations to obtain the corresponding runes: alpha, beta, and gamma. The sequence and results of reaching these runes in the correct order were as follows:

`alpha` was located at (73, 82)

![](https://i.imgur.com/l2EDQzq.png)

`beta` was located at (76, 80)

![](https://i.imgur.com/WkeFKuM.png)

`gamma` was located at (89, 69)

![](https://i.imgur.com/AeeNBZt.png)

## Methodology
Throughout this challenge, I relied solely on Notepad++ for modifying the source code to facilitate sanity checks. I did refer to some functions less familiar to me, conducting occasional research. All other tools used were integrated into Chrome's dev tools panel.

### Step 1: Inspection
The initial step involved checking the prompt for any concealed messages. After determining that none were present, I proceeded to examine the source code using Chrome's inspect tool. Upon doing so, I encountered a seemingly stuck debugger cue, which I interpreted as part of a deliberate attempt to hinder my progress in finding the treasure.

Navigating through the script, I observed numerous defined functions and built-in functions. I made a mental note of these variables and functions before closing the inspect panel in Chrome. Subsequently, I decided to interact with the live page to uncover the treasure. Noticing the X marker, I began clicking around it until it responded, realizing that I could manipulate the X marker by dragging it to specific locations on the page, akin to a pirate navigating maps. 

### Step 2: Decoding
In the initial phases of scrutinizing the script, I encountered a considerable amount of intricate code assigned to variables like `spikes` and `ee`. Notably, I observed the assignment `b = atob`, prompting me to suspect that `atob` might be a built-in function. A swift Google search confirmed my suspicion, revealing that `atob()` serves as a function for decoding Base64-encoded strings.

Armed with this knowledge, I applied the `atob` function, linked to the variable `b`, to various applicable elements. I decoded all the strings within the `spikes` array, the content of the `ee` variable, and what appeared to be the win alert on line 75. The following is a compilation of the successfully decoded items:

- Line 36 - `spikes = ['navigator', 'userAgent', 'indexOf', 'LAMBERT', 'RIPLEY', 'ALDERSON', 'BLACKSEA', 'dir', 'clear', 'clue', 'canary', 'alpha', 'beta', 'gamma']`

- Line 75 - `eval('alert("Congratulations Oh 1337 One! You have found my treasure!")` 

While attempting to decode the entirety of the content, I observed that certain items enveloped within the `b` function exhibited a nested `tY` function. In my effort to comprehend the functionality of the `tY` function, I deduced that it essentially interchanged the first and last characters of the parameter or string. Armed with this newfound insight, I successfully deciphered additional sections of the code:

- Line 47 - `window['navigator']['userAgent'].indexOf('BLACKSEA') == 9`

- Line 48 - `‘RIPLEY’`

- Line 50 - `‘LAMBERT’`

- Line 57 - `‘ALDERSON’`

I didn’t know what they meant just yet, but things were about to get interesting.

### Step 3 - Win conditions
Upon delving deeper into the code, I scrutinized the if statement preceding the win alert, revealing that triggering the alert hinged upon satisfying two distinct conditions. Firstly, the `runes` array must contain more than two elements. Secondly, the value within the `clue` variable must align with the parameters of an unnamed function, denoted as `targ`. These conditions collectively enable the win alert based on the prior decodings.

The exclusive function responsible for appending elements to the `runes` array is the `IChing` function, which takes x and y coordinates as parameters. Analyzing the function, it became evident that the runes-alpha, beta, and gamma-would be added using the `.push` method when the X marker landed on specific spots on the page. Intriguingly, these spots must be activated in a particular sequence, aligning with the order of compound conditional statements on lines 87, 89, and 91. This deduction arose from the `runes.length` condition within each compound statement, indicating a prerequisite `runes` length to add an element, thus satisfying the condition for the subsequent statement.

Within each compound statement, the y and x variables necessitate correspondence with the ASCII representation of distinct characters in the string assigned to the `clue` variable. Hence, it is imperative to have the correct value assigned to `clue`. The initialization of the `clue` variable allows for two values: `‘RIPLEY’` and `‘LAMBERT’`. Building on previous deductions, assigning `‘RIPLEY’` to the `clue` variable is paramount for acquiring the accurate x, y coordinates to locate the treasure. Conversely, assigning `‘LAMBERT’` to `clue` within the `IChing` function not only yields inaccurate coordinates but also violates the second of the two previously stated conditions.

> [!IMPORTANT]
> Ensuring precision, we verify that the variable `clue` holds the accurate value, unveiling the precise coordinates to navigate with the X marker.

### Step 4: Finding the right clue
Upon decoding the entire `clue` variable with the `atob` function, the function's workings became more apparent. Employing an educated guess, I deduced that the variable `clue` need to contain the first return statement of the if statement, given that the else statement served as the consistent default. This inference was subsequently validated by inspecting the side panel of the dev tools in Chrome.

![](https://i.imgur.com/BfGrQex.png)

The focus shifted to pinpointing the condition that results in the assignment of `‘RIPLEY’` to the variable `clue`, as it corresponds to the initial return statement in the if statement. Examining the condition necessary for this occurrence—the first line of the function—it appeared as follows: `window['navigator']['userAgent'].indexOf('BLACKSEA') == 9`. 

Through a bit of exploration on Google, I uncovered that the `window` object houses a diverse range of functions, namespaces, objects, and constructors. These elements aren't exclusively tied to the concept of a user interface window, suggesting a broader browser-related association rather than one specific to the UI. Following this trail, I delved into the meanings of `navigator` and `userAgent`. Voilà! It became evident that `navigator` is an attribute of the `window` object, and `userAgent` is an attribute of the `navigator` object.

This realization prompted me to understand that the user agent, a customizable attribute within the Chrome browser, could be altered. I proceeded to change the browser's user agent to initiate the reading of `'BLACKSEA'` at the 9th index, aligning with the line of code. This adjustment ensured that the variable `clue` was now assigned the value `‘RIPLEY’`. 

![](https://i.imgur.com/pVhzJoa.png)

With the correct clue in position, we can now proceed to reveal the treasure. 

### Step 5: Putting it all together
The moment had arrived to set sail across the seas. However, a hurdle presented itself-the persistent debugger prompt hindered real-time movement of the marker, disrupting my navigation. It was imperative to gain access to the console for live coordination viewing as I manipulated the marker. Consequently, I downloaded the HTML file and introduced a few enhancements for a smoother experience using Notepad++. Here is an overview of the adjustments made:

- I had removed `ee('dLb'.replace('L', 'e') +'ug' + '14r'.replace('14', 'ge'))` from the code in order for me to move the marker in real-time while viewing the console.

- I had removed `console.dir(el)` since I did not need to view the properties at this time.

- I had removed `console.clear()` from the code in order to prevent it from clearing the entries in the console.

The latter two items I had removed were also evident in the spikes array, suggesting a potential need for future adjustments.

One of the last challenges I addressed involved obtaining real-time coordinates of the marker, a feature not supported by the source code. While examining the div tag with the id of `‘gps’`, I recognized its role in displaying the encoded representation of the current coordinates on the screen: `document.getElementById("gps").innerHTML = btoa(y.toString() + "," + x.toString()).replace("=", "\n\<br>\\n")`. However, this representation isn’t useful.

In addition to the inner HTML provided by the original source code, I incorporated a display of the x and y coordinates into the user interface, offering a clear view of the X marker's location. With this newfound capability to identify the marker's coordinates, the next step involves aligning them with the ASCII representation of the characters in the variable assigned to `clue` (`‘RIPLEY’`). Leveraging the `.charCodeAt` method and referencing an ASCII table, I obtained the sequential coordinates necessary to traverse (as dictated by the `IChing` function's compound conditional statements) and reach the ultimate destination—a culmination of the win condition outlined in step 3.

> [!NOTE]
> The coordinates presented in the compound conditional statements within the `IChing` function were structured with the y-coordinate listed first, followed by the x-coordinate. This detail holds significance because it could have been misleading if overlooked, given that conventional coordinate notation typically places the x-coordinate first and the y-coordinate second.

These were the final deduced coordinates:

- ASCII (I) = 73, ASCII (R) = 82 - **First rune location (73, 82)**
- ASCII (L) = 76, ASCII (P) = 80 - **Second rune location (76,80)**
- ASCII (Y) = 89, ASCII (E) = 69 - **Third rune location (89,69)**

Navigating to these predetermined locations with the assistance of my real-time coordinate line of code, I observed the dynamic population of the `runes` array—another code modification I implemented for real-time visibility. Upon reaching the ultimate position, the win alert promptly surfaced, affirming the successful discovery of the treasure!


