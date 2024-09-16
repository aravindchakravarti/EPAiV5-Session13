# Session 13 Assignment

### Submission Details:
Name: Aravind D. Chakravarti

Email: aravinddcsadguru@gmail.com


# About Context Managers in Python

Context Managers in Python
Context managers handle setup and cleanup tasks (e.g., opening and closing files) using with statements, ensuring proper resource management.

## 1. Class-Based Context Managers

We define context managers using a class with `__enter__` and `__exit__` methods:

`__enter__`: Executed at the start of the with block. It can return a resource (like a file handle).
`__exit__`: Executed when the with block is exited, handling cleanup. It accepts exceptions if raised inside the block.

In the session file, we have a class named `class CSVDataIterator:` which implements both `__enter__` and `__exit__` methods as follows. 

```python
 def __enter__(self):
        ''' When enter the context, read the first line (which is header) and return the object
        '''
        try:
            self._fd = open(self._file, self._mode)
            self._CSV_file_fd = csv.reader(self._fd, delimiter= self._separator)
            self._header = list(next(self._CSV_file_fd))
            self._csv_data = namedtuple('csv_data', self._header)
            return self
        except IOError as e:
            print(f"Error opening file: {e}")
            raise
    
    def __exit__(self, exc_type, exc_value, exc_tb):
        ''' Upon exiting the context close the file and display any error/exception occurred. 
        if exception has occured then stop the excecution of the code
        '''
        self._fd.close()
        if exc_type:
            print(f"Exception {exc_type} occurred: {exc_value}")
            
        if self._fd.closed:
            print("File closed properly")
        else:
            print("File did not close properly.")
        return False
```

First we open the file and if successful, we pass the file handle to the `CSV` reader. `CSV` reader returns the generator, which we store it in `self._CSV_file_fd` and use it in our `__next__` method. In this way, we are using context-manager to generate the data on a `CSV` file. 


## 2. Contextlib-Based Context Managers

We can also create context managers using the `@contextmanager` decorator from the `contextlib` module. This involves a generator function with a `try`...`finally` block.

`yield`: Separates setup (try block) from cleanup (finally block). The part before yield runs on entering the `with` block, and the part after runs when exiting.

```python
@contextmanager
def CSVDataIterator_f(fname :str, mode :str = 'r', separator :str= ','):
    ''' Function helps in generating the "Generator" by reading a csv file.
    '''
    try:
        file_fd = open(fname, mode)
        csv_file_fd = csv.reader(file_fd, delimiter=separator)
        # Read the header explicitly
        header = next(csv_file_fd)

        # yield untill end of the file 
        yield csv_file_fd
    except FileNotFoundError:
        print("No such file exists in this path")
    finally:
        file_fd.close()
```

In the above function we are using `@contextmanager` decorator, which helps us in adding `__enter__` and `__exit__` method automatically. We decorate our `CSVDataIterator_f` which implements the generator. When the code enters the `with` block, everythig from `open` up to `yield` gets executed. `yield` gets executed whenever we call `for` loop. When we hit `stopiteration` exception, we execute the `finally` block to close the file.
