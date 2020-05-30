5/25:

Task 0: bug fix

remove "delete _chatBot".

There is no need to over free memory here. The pointer is parsed in and should be deleted in the orginal class where it was created.

5/26:

Task 1: unique_ptr

In chatgui.h file, I converted the raw pointer _chatLogic , to std::unique_ptr;

in chatgui.cpp file also, I changed it to std::uniqu_ptr using std::make_unique and made necessary changes in the destructor.

But then I'm getting the following error while compiling:

```
error: no viable conversion from returned value of type 'std::unique_ptr<ChatLogic>' to function
      return type 'ChatLogic *'
    ChatLogic *GetChatLogicHandle() { return _chatLogic; }
                                             ^~~~~~~~~~
1 error generated.
```

Then I simply change the return type of the method.

```
    std::unique_ptr<ChatLogic> GetChatLogicHandle() { return _chatLogic; }
```

Then a new error happened:

```
error: call to implicitly-deleted copy constructor of 'std::unique_ptr<ChatLogic>'
    std::unique_ptr<ChatLogic> GetChatLogicHandle() { return _chatLogic; }
                                                             ^~~~~~~~~~
/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/../include/c++/v1/memory:2493:3: note: copy constructor is implicitly deleted
      because 'unique_ptr<ChatLogic, std::__1::default_delete<ChatLogic> >' has a user-declared move constructor
  unique_ptr(unique_ptr&& __u) noexcept
  ^
1 error generated.
```

As a unique_ptr natured, its copy constructor has been deleted implicitely. The value copy behavior like this is forbbiden.

There is a good answer about this:

Unique pointers do not have copy constructor or copy assignment operator. They have been deleted using the 'delete' operator in its declaration. Why? Because the very nature of unique pointer is, unique ownership. At a time there should be only 1 pointer managing the resource in heap. So unique pointers only have move constructor and move assignment operator. That way they can only be moved and never copied.

In the GetChatLogicHandle function, it was trying to return a unique pointer by value. This prompts the compiler to look for the unique_ptr's copy constructor which we now know is deleted. Hence the error message "use of deleted function" showed up. If you check the function signature in the error message you will see it's the signature of a copy constructor.

When you use std::move operation on _chatLogic to get around the "use of deleted function" error, what's happening is,

a) std::move will prompt the compiler to invoke move constructor of std::unique_ptr which is available. Hence it won't throw error.

b) However, when you performed std::move on _chatLogic, you effectively transferred the ownership of that heap resource to your new return variable. Now the member variable _chatLogic no longer has the ownership of the heap resource. That's why your program was crashing.

When you call get() method on a smart pointer, it returns the address of the memory location in heap that it's managing. So when you return the _chatLogic.get() value, you are just returning a copy of the memory address by value. That way the _chatLogic unique_ptr doesn't get invalidated and the program won't crash.

5/29:

Task 2 : The Rule Of Five

In chatbot.h & chatbot.cpp:

add in:

  - copy constructor
  - move constructor
  - copy assignment
  - move assignment

.

With

  - destructor

which was already in there, they together are concluded as Rule of Five.

Reference: https://en.cppreference.com/w/cpp/language/rule_of_three

Task 3: Exclusive Ownership 2

A vector of unique_ptr applied on _nodes.

Task 4: Moving Smart Pointers

Move Ownership of GraphEdge

```
std::unique_ptr<GraphEdge> edge = std::make_unique<GraphEdge>(id);
```

Task 5: Moving the ChatBot

TBD
