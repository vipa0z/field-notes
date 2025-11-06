### If you can upload files

- Uploading a small PHP file that calls `phpinfo()` is the simplest path _if uploads can be executed as PHP_.
```

<?php
 phpinfo();
  ?>
```
also  a php script to upload to the vulnerable server and determine disabled functions (from ippsec's hospital video)
`/tools/disabled_functions_checker.php`
```
<?php    
$dangerous_functions = array('pcntl_alarm','pcntl_fork','pcntl_waitpid','pcntl_wait','pcntl_wifexited','pcntl_wifstopped','pcntl_wifsignaled','pcntl_wifcont  
inued','pcntl_wexitstatus','pcntl_wtermsig','pcntl_wstopsig','pcntl_signal','pcntl_signal_get_handler','pcntl_signal_dispatch','pcntl_get_last_error','pcntl  
_strerror','pcntl_sigprocmask','pcntl_sigwaitinfo','pcntl_sigtimedwait','pcntl_exec','pcntl_getpriority','pcntl_setpriority','pcntl_async_signals','error_lo  
g','system','exec','shell_exec','popen','proc_open','passthru','link','symlink','syslog','ld','mail');  
foreach($dangerous_functions as $f) {  
   if (function_exists($f)) {  
       echo $f . " exists<br>\n";  
   }  
}  
  
?>
```

search google:  php <popen> run command or simillar depending on the output.
using popen
![[Pasted image 20251005143709.png]]


