### C++11 Move Iterator
there is move iterator in C++ where you can append one container items into another by moving them one by one, taking the iterator that only returns the r-value
reference.

to make a move iterator: `std::make_move_iterator(container.begin())`
