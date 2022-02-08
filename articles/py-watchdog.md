## Python 库 - watchdog

库地址：[pypi](https://pypi.org/project/watchdog/) [github](https://github.com/gorakhargosh/watchdog)

库功能：文件监控

使用示例：

```
from watchdog.events import FileSystemEventHandler
from watchdog.observers import Observer

class FileEventHandler(FileSystemEventHandler):
    
    def __init__(self, callback_func):
        super(FileEventHandler, self).__init__()
        self.callback_func = callback_func
	
	def on_moved(self, event):
        what = 'directory' if event.is_directory else 'file'
        self.callback_func(src_path, what)

    def on_created(self, event):
        what = 'directory' if event.is_directory else 'file'
        self.callback_func(src_path, what)

    def on_deleted(self, event):
        what = 'directory' if event.is_directory else 'file'
        self.callback_func(src_path, what)

    def on_modified(self, event):
        what = 'directory' if event.is_directory else 'file'
        self.callback_func(src_path, what)


class TaskHandler(object):

    def __init__(self):
        self.file_observer = None
        self.file_observer_path = None

    def observer_start(self):
        if not os.path.exists(self.file_observer_path):
            os.makedirs(self.file_observer_path)
        event_handler = FileEventHandler(self.callback)
        self.file_observer = Observer()
        self.file_observer.schedule(event_handler, self.file_observer_path)
        self.file_observer.start()

    def observer_stop(self):
        if self.file_observer:
            self.file_observer.stop()
            self.file_observer.join()
    
    def observer_restart(self):
        self.observer_stop()
        self.observer_start()

    def callback(self, src_path):
        self.logger.debug('file changed. {}'.format(src_path))
        
    def run(self):
        self.observer_start()
```

