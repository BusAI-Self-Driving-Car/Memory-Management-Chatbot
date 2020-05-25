5/25: Task 0: bug fix
    remove "delete _chatBot;". There is no need to over free memory here.
    The pointer is parsed in and should be deleted in the orginal class where it was created.


