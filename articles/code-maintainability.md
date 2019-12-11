# Improving code maintainability

The other day I had a discussion about how to make code more maintainable. Someone was convinced that simplicity and documentation are the ways to get there.

I believe that is not the case. Although I am a fan of the 
[YAGNI](https://martinfowler.com/bliki/Yagni.html) and [KISS](https://en.wikipedia.org/wiki/KISS_principle) principles as mentioned in 
[C# Coding Guidelines](https://csharpcodingguidelines.com/), in my opinion software becomes more maintainable by trading 
simplicity for complexity and improving code readability instead of writing documentation.

But first, what do we mean by code maintainability? The [IEEE Standards Association](https://ieeexplore.ieee.org/document/7321183) defines is as:

> The ease with which a software system or component can be modified to correct faults, improve performance or other attributes, or adapt to a changed environment.

Makes sense. Ok, let's move on.

## Trading simplicity for complexity

Allow me to illustrate this with an example. Let's assume we need to write a program that prints "Hello, Jack" five times. 
The simplest solution would be:

```csharp
Console.WriteLine("Hello, Jack");
Console.WriteLine("Hello, Jack");
Console.WriteLine("Hello, Jack");
Console.WriteLine("Hello, Jack");
Console.WriteLine("Hello, Jack");
```

Now let's consider the maintainability of this solution for a moment. There is a lot of duplication, which is bad. 
Because if we need to change "Jack" to "Alice", then we need to change all the lines. 
Furthermore, it takes a bit of effort to see that the number of lines printed is 5 instead of 4 or 6. We can do better!

```csharp
for (int count = 0; count < 5; count++)
{
    Console.WriteLine("Hello, Jack");
}
```

Now we have reduced the number of lines and removed the duplication. Changing "Jack" to "Alice" is a one-liner now. 
And it is easy to spot that the code prints 5 lines. But we have added a lot of complexity. 
There is now a looping construct. And we have introduced a counter variable. And a scope block. 
Still, this change has improved maintainability, which was our goal, so we are on the right track.

As software evolves, usually new requirements emerge. Let's assume the name to print changes from time to time. The simplest solution
would be to change our source code each time and recompile. Another solution is to adapt our code to work for varying names 
by allowing the user to type it in:

```csharp
Console.WriteLine("Please type the name and press [Enter].");
string name = Console.ReadLine();

for (int count = 0; count < 5; count++)
{
    Console.WriteLine("Hello, " + name);
}
```

Again, we added complexity. We now have a second variable that the reader needs to keep track of. And we are now using the '+' operator 
to concatenate pieces of text. The reader now has to be familiar with that syntax too. But all this complexity was added to achieve 
more flexibility. Because we know that requests for name changes will keep coming and our code changes will remove the need
for us to make changes to the code all the time.

To further improve maintainability, we can consider the [Single Responsibility](https://en.wikipedia.org/wiki/SOLID) principle. 
It means that each piece of code should have only one responsibility or only one reason to change.
In our case, we have three concerns. Reading input, looping and writing output. Let's separate them out.

```csharp
string name = GetNameFromUser();
PrintNameMultipleTimes(name);

string GetNameFromUser()
{
    Console.WriteLine("Please type the name and press [Enter].");
    return Console.ReadLine();
}

void PrintNameMultipleTimes(string name)
{
    for (int count = 0; count < 5; count++)
    {
        PrintName(name);
    }
}

void PrintName(string name)
{
    Console.WriteLine("Hello, " + name);
}
```

Again, lots of extra complexity was added. And now we even have more code than before! This must be a bad change, right? 
Well, the good thing is that reading the entire source is no longer required in most cases. If a reader wants to understand the
details on the printing, its easy to skip over the input processing because it was extracted out. As programs grow in time, it 
becomes key to keep separating concerns out in order to keep a mental model that fits in our brain. So not too many variables, scopes,
features (implementation details) at the same place. 

This all has to do with abstractions. Just like a device driver in your operating system presents to you a file system with files 
and directories (hiding the mechanical details of the disk), software building blocks like functions and classes represent concepts which are
generally understood by humans without needing to know all the details. Hiding such details decreases coupling, which makes it easier 
to swap out parts without affecting the rest. Which improves maintainability too.

I think I've makde my point on simplicity. We need to add complexity (smarter constructs), not simplicity, in order to deal with 
emerging requirements while keeping the codebase maintainable.

## Documentation

Let's assume that we are building a software library that is used by others.

For example, consider the next documented method:
```csharp
/// <summary>
/// Prints a welcome text for the specified user.
/// </summary>
void Print(string name)
{
    Console.WriteLine("Welcome, " + name);
}
```

At a later time, someone changes the implementation:

```csharp
/// <summary>
/// Prints a welcome text for the specified user.
/// </summary>
void Print(string name)
{
    Console.WriteLine("User " + name + " has been added to the mailing list.");
}
```

Did you spot how the documentation has gotten out of sync? In practice, this happens all the time. A better approach would be to name the method `PrintWelcomeTextForUser` initially, which should then be changed to `PrintUserAddedToMaillingList`. This is still not a guarantee, but renaming the method reveals the intention change at the call side. The diff in the PR would highlight all places that affect this change, making it easier to determine impact.

So writing documentation decreases *our* maintainability. Whether it is an explaining comment in the code, a usage guide or a wiki, 
over time they tend to deteriorate because we forget to keep them up-to-date when code changes. Then they cause confusion instead 
of clarity, because there are now two versions of the truth. The code and the documentation. But in the end, the code is always 
telling the truth. So it is better to invest in proper naming, resulting in readable and maintainable code, than in documenting 
all the peculiarities.

From the IEEE definition, we get that maintainable software means that code is easy to adapt. Having numerous documents that need 
to be kept in sync manually makes adapting take more effort, so it decreases the maintainability.

However, us writing documentation benefits the consumers of our library greatly. Since they usually do not have access to our 
source code, they depend on documentation and trial-and-error to figure out how things work. And us documenting what has changed 
in the next version enables them to plan migration with less risk and focus their testing efforts. I would advice to invest in
automating the generation of documentation when possible, to provide consistency and require minimal future effort.

Thanks for reading!
