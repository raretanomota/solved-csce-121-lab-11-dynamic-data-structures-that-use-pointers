Download Link: https://assignmentchef.com/product/solved-csce-121-lab-11-dynamic-data-structures-that-use-pointers
<br>
The purpose of this lab is to give some practice in using dynamic data structures that use pointers to organize access. It also introduces a few other ideas along the way, such as C++ strings and the stringstream.

This lab has quite a bit of code provided upfront. It is a good idea to download, compile and play with it before attending your lab session. Some of the questions in the “More interesting operations” build on earlier pieces, while others are quite separate. If you get stymied with some especially elusive bug, consider reading the next couple of bullet points. If they don’t obviously depend on your current part, you might be able to make progress and return to your debugging with fresh insight. Remember both valgrind (for memory leaks) and gdb (for segfaults) can help in the hunt for problems.

You may need to enable the C++11 options via the following flag: “g++ -std=c++11”

<h2>All creatures great and small</h2>

<h3>Overview</h3>

This week, we’ll be storing data to describe nature conservation areas, animal sanctuaries, wildlife refuges, and zoos. All these places (and their associated organizations) manage animals. Consequently, we’ll need an entity to represent an animal. Here’s one that, though perhaps not terribly detailed, is a starting point:

struct animal_t {    string desc; // A description of the animal    int age; // Age of the animal    bool is_nocturnal; // True if the animal is nocturnal     animal_t *prev; // Pointer to previous animal in the list    animal_t *next; // Pointer to the next animal in the list};

The image on the right shows that it is useful to think of the structure in terms of fields representing data specific to the animal and pointers serving to help organize the data. There are two additional things worth noting:

<ol>

 <li>A description of the animal is stored in a field named desc and it is declared to be of type <a href="http://www.cplusplus.com/reference/string/string/">string</a>. In order to use this type, you’ll need to have #include &lt;string&gt;  at the top of your source code. The strings are a modern C++ type that can be much more convenient for textual data than C’s char[]s because they can be resized easily. They call new and delete internally as needed themselves. They also “know” their own length, so are significantly safer. Standard operations like assignment (=) and comparison (&gt;=, &lt;=, ==, etc.) work as you might expect. You’re also still able to get the <a href="http://www.cplusplus.com/reference/string/string/c_str/">underlying char data</a> if you want it.</li>

 <li>Unlike the queue used last week, we’re going to keep two pointers between elements of our list. It’ll be slightly more bookkeeping, but will make some things quite a lot easier. The pizza queue was a singly-linked list and this is going to be a doubly-linked list (also sometimes called a double linked list).</li>

</ol>

It is also helpful to have a separate structure to describe facilities and their particular data. Again, the example code is fairly parsimonious, but it gives the idea:

struct facility_t {    string name; // Name of the facility     int animal_count; // A count of the elements in the linked list    animal_t *head; // Head of the doubly-linked list    animal_t *tail; // Tail of the doubly-linked list};

The spacing here is intended to make apparent the fact that there’s a block for managing the list of animals and that it is logically distinct from the other bits. We’ll have the animals associated with a particular facility via pointers. As the following diagram shows, we’ll keep track of both ends of the list:

<h3>Getting up and running</h3>

Since it can be rather tedious to populate these lists, especially each time you want to test some particular case, I’ve provided some code that will initialize a doubly-linked list via data loaded from a text file. Here’s the code:

<ul>

 <li>Starting code: <a href="http://robotics.cs.tamu.edu/dshell/cs121/l11/zoo-starting-code.cpp">C++ Source</a></li>

 <li>Data files: <a href="http://robotics.cs.tamu.edu/dshell/cs121/l11/escobar.txt">Escobar’s Menagerie</a>, <a href="http://robotics.cs.tamu.edu/dshell/cs121/l11/central_park_zoo.txt">Central Park Zoo</a>, <a href="http://robotics.cs.tamu.edu/dshell/cs121/l11/mimis.txt">Mimi’s Marsupials</a>.</li>

</ul>

Inspect the data files, then compile, run, and play with the code. You’ll see that we have a convenient way to initialize multiple facilities. For example, among others, it’ll create this one:

Trace through the create_facility function. You’ll see that it uses the string type extensively. That is because this version of getline doesn’t need to be given a maximum size: the string grows as needed. The function also makes use of a new stream called stringstream which is convenient because it allows one to use the &gt;&gt; operator on a string, just as if it were a file or cin. (Did you spot that new header file in the preprocessor include at the top of the source?)

Look at the add_animal_to_facility_head function to see the doubly-linked list in action. You might find it useful to draw a little diagram to see it working. For example, starting from an empty list add three animals, one after the other.

<h3>Functionality to implement</h3>

Two functions use new to allocate memory. We know they ought to have matching functions to free that memory (via delete), but those weren’t provided. Here are some steps that take you through fixing that issue.

<ul>

 <li><em>Disconnecting a list element.</em> <strong>Step 1</strong>: define a function to unlink a given animal from the list.</li>

</ul>

<ul>

 <li>voidunlink_from_inventory(…) {       … // After this call, the animal_t should exist  ·       … // but no longer be part of the doubly-linked list·     }</li>

</ul>

First determine what arguments it should take.

<table>

 <tbody>

  <tr>

   <td>

    <table>

     <tbody>

      <tr>

       <td><strong>?</strong></td>

       <td>HINT</td>

      </tr>

     </tbody>

    </table></td>

   <td>

    <table>

     <tbody>

      <tr>

       <td>You have to pass it a pointer to the animal_t that is to be unlinked. That much is obvious. But you also have to pass it a pointer to the facility_t because animal_count must be decremented and head, or tail, or both may need to be updated.</td>

      </tr>

     </tbody>

    </table></td>

  </tr>

 </tbody>

</table>

Then implement it. You can test it on the following case:

The US and China have had a spat. China demands its panda back. Make a facility for the Beijing Zoo, then use find_animal to get a pointer to the panda. Remove it from the Central Park Zoo, via unlink_from_inventory, and then add it to Beijing (via add_animal_to_facility_head).

<ul>

 <li><em>Freeing the memory associated with a list element.</em> <strong>Step 2</strong>: write a function to delete the animal. Note that after calling this function, all pointers previously referring to that item should now be considered invalid.</li>

</ul>

<ul>

 <li>voidfree_animal_mem(…) {       … // After this call, no more memory is associated with the given animal_t·     }</li>

</ul>

<ul>

 <li><em>Freeing the memory associated with a whole list.</em> <strong>Step 3</strong>: assuming that animals never appear in more than one list, facilities should delete their associated lists before being deleted themselves. Write a function that does this.</li>

</ul>

<ul>

 <li>voidfree_facility_mem(…) {       … // After this call, no more memory is associated with the given facility_t·     }</li>

</ul>

Your implementation should call the previous two functions. Call your function at the end of your program for each facility that you’ve created.

At this point, if your functions are working correctly, you should be able to run valgrind and get a clean report showing no memory leaks.

<u>An important reflection</u>: The function free_animal_mem was very straightforward. That’s because the animals themselves didn’t have anything (i.e., any fields/values) dynamically allocated. It meant that free_animal_mem was little more than a call to delete. In contrast, free_facility_mem had real work to do. At first sight, it might’ve seemed silly to even have a function free_animal_mem. But consider that if we do create some new dynamic variable(s) for the animals (maybe in add_new_animal_to_inventory) we now have a place to relinquish that memory because free_facility_mem calls free_animal_mem. A function that relinquishes resources on the destruction of a data structure (resources like dynamic memory that was allocated, or closing files that were opened, etc.) is called a <em>destructor</em>.

<h3>More interesting operations</h3>

There are several interesting operations worth implementing with doubly-linked lists.

<ul>

 <li><em>Adding items.</em> Given a reference to a facility_t and an animal_t within that facility’s inventory, write a function that adds another animal_t into the inventory so that it appears directly <em>after</em> the given animal_t. In the picture below, the arguments were the Central Park Zoo and the boomslang. The dromedary was added.Now add a fourth argument  bool add_after=true  to the argument list of your function. (The = true gives a default value so you can still call it with only three parameters. Cool huh?) Improve your function so that when the argument add_after is false, the new animal is added <em>before</em> the other one. Be careful to ensure that you maintain the validity of both the head and tail.</li>

 <li><em>A selective query.</em> Write a function that, given a facility, prints out the list of the animals you’re likely to see being active on a nighttime visit.</li>

 <li><em>Transferring lists.</em> Escobar’s Menagerie was raided by the DEA’s office and the animals seized. They are to be transferred to the Central Park Zoo. Write a function that, given two facility_ts, results in the items in the second list being concatenated with the items in the first list. Afterwards, the first list should have all the items and the second list should be empty.</li>

 <li><em>A linked list masquerading as an array.</em> Write a function, get_nth, to get the n<sup>th</sup> item in a linked list. The language python has a cool feature where you can use negative numbers to count backward from the end of a list. Co-opt that feature by including this behavior in your get_nth function as well.</li>

 <li><em>A linked list masquerading as a better array.</em> Write a function that allows you add an item into the list at location n. You’ll have to check that n makes sense.</li>

 <li><em>Splitting an item.</em> Write a function that allows you to split an item in a list into two. Test it on the following case:A particular facility has some pretty meagre animal offerings. Among their catalog for public display they count an earthworm. Zoo keeper Vince observes only one earthworm in the worm enclosure on Monday afternoon. But on Tuesday, a more diligent employee (Howard) spots two.To ensure that after your function you do actually have two entirely distinct earthworms, update the string for the second earthworm to read earthworm’ (that’s a prime). Then check that the first’s name has remained unchanged.</li>

 <li><em>Cloning.</em> Unbeknownst to the zoo management, disgruntled employees have been working late into the night conducting wicked experiments. Their project is to open a competing facility that’ll charge lower entrance fees. They want to ensure that they have all the same animals, so have been engaged in cloning individual animals. Write a function that, given a pointer to a facility_t, will produce another one with copies of the same animals.The crux here is that you need to clone the animals, not just the facility. This is sometimes called a <em>deep copy</em> — in contradistinction to a <em>shallow copy</em>.(Another exercise, which is also useful practice if you’re still wanting a bit more, is to write a function that clones the facility but only keeping the daytime animals; those are the ones that attract the top dollar from visitors anyway.)</li>

</ul>

<h3>Ordered insertion*</h3>

As mentioned above, the string type allows comparison of strings via operations like &lt;= and &gt;=. It orders strings lexicographically, i.e., alphabetically with ties broken in favor of the shorter string. This is the way words are arranged in a dictionary. Using this feature, write the following function:

void add_animal_to_facility_sorted(facility_t *facility, animal_t *ani){  … // place animals in position to maintain a sorted inventory}

By replacing the last line of add_new_animal_to_inventory so that it calls this function instead, you can test your code easily by rearranging the animals within the input files.