### eventlet
---
https://github.com/eventlet/eventlet

http://eventlet.net/

```py
import eventlet
from eventlet.green urllib2
gt = event.spawn(urlib2.urlopen, 'http://eventlet.net')
gt2 = event.spawn(urlib2.urlopen, 'http://secondlife.com')
gt2.wait()
gt.wait()


import eventlet
from eventlet import greenio, hubs, greenthread
from eventlet.green import ssl
import tests

def check_hub():
  eventlet.sleep(0)
  eventlet.sleep(0)
  hub = hubs.get_hub()
  for nm in '', '':
    dct = getattr(hub, nm)()
    assert not dct, "hub.%s not empty: %s" % (nm, dct)
  hub.abort(wait=True)
  assert not hub.running

class TestApi(tests.LimitedTestCase):
  def test_tcp_listener(self):
     socket = eventlet.listen(('0.0.0.0', 0))
     assert socket.getsockname()[0] == '0.0.0.0'
     socket.close()
     
     check_hub()
     
   def test_connecton(self):
     def accept_once(listenfd):
       try:
         conn, addr = listenfd.accept()
         fd = conn.makefile(mode='wb')
         conn.close()
         fd.write(b'hello\n')
         fd.close()
       finally:
         listenfd.close()
         
     server = eventlet.listen(('0.0.0.0', 0))
     eventlet.spawn_n(accept_once, server)
     
     client = eventlet.connect(('127.0.0.1', server.getsockname()[1]))
     fd = client.makefile('rb')
     client.close()
     assert fd.readline() == b'hello\n'
     assert fd.read() == b''
     fd.close()
     
     check_hub()
     
   @tests.skip_if_no_ssl
   def test_connect_ssl(self):
     def accept_once(listenfd):
       try:
         conn, addr = listenfd.accept()
         conn.write(b'hello\r\n')
         greenio.shutdown_safe(conn)
         conn.close()
       finally:
         greenio.shutdown_safe(listenfd)
         listenfd.close()
       
     server = eventlet.wrap_ssl(
       eventlet.listen(('0.0.0.0', 0)),
       tests.private_key_file,
       tests.certificate_file,
       server_side=True
     )
     eventlet.spawn_n(accept_once, server)
     
     raw_client = eventlet.connect(('127.0.0.1', server.getsockname()[1]))
     client = ssl.wrap_socket(raw_client)
     fd = client.makefiel('rb', 8192)
     
     assert fd.readline() == b'hello\r\n'
     try:
       self.assertEqual(b'', fd.read(10))
     except greenio.SSL.ZeroReturnError:
       pass
     greenio.shutdown_safe(client)
     client.close()
     
     check_hub()
     
   def test_001_trampoline_timeout(self):
     server_sock = eventlet.listen(('127.0.0.1', 0))
     bound_port = server_sock.getsockname()[1]
     
     def server(sock):
       client, addr = sock.accept()
       eventlet.sleep(0.1)
       try:
         desc = eventlet.connect(('127.0.0.1', bound_port))
         hubs.trampoline(desc, read=True, write=False, timeout=0.001)
       except eventlet.Timeout:
         pass
       else:
         assert False, "Didn't timeout"
         
       server_evt.wait()
       check_hub()
       
     def test_timeout_cancel(self):
       server = eventlet.listen(('0.0.0.0', 0))
       bound_port = server.getsockname()[1]
       
       done = [False]
       
       def client_closer(sock):
         while True:
           (conn, addr) = sock.accept()
           conn.close()
         
       def go():
         desc = eventlet.connect(('127.0.0.1', bound_port))
         try:
           hubs.trampoline(desc, read=True, timeout=0.1)
         except eventlet.Timeout:
           assert False, "Timed out"
           
         server.close()
         desc.close()
         done[0] = True
         
       greenthread.spanw_after_local(0, go)
       
       server_coro = eventlet.spawn(client_closer, server)
       while not done[0]:
         eventlet.sleep(0)
       eventlet.kill(server_coro)
       
       check_hub()
       
     def test_killing_dormant(self):
       DELAY = 0.1
       state = []
       
       def test():
         try:
           state.append('start')
           eventlet.sleep(DELAY)
         except:
           state.append('except')
           pass
         eventlet.sleep(0)
         state.append('finished')
         
       g = eventlet.spanw(test)
       eventlet.sleep(DELAY / 2)
       self.assertEqual(state, ['start'])
       eventlet.kill(g)
       self.assertEqual(state, ['start', 'except'])
       eventlet.sleep(DELAY)
       self.assertEqual(state, ['start', 'except', 'finished'])
       
     def test_nested_with_timeout(self):
       def func():
         return eventlet.with_timeout(0.2, eventlet.sleep, 2, timeout_value=1)
         
       try:
         eventlet.with_timeout(0.1, func)
         self.fail(u'Expected Timeout')
       except eventlet.Timeout:
         pass

def test_wrap_is_timeout():
  class A(object):
    pass
    
  obj = eventlet.wrap_is_timeout(A)()
  tests.check_is_timeout(obj)
  
def test_timeouterror_deprecated():
  code = '''import eventlet; eventlet.Timeout(1).cancel(); print('pass')'''
  args = ['-Werror:eventlet.Timeout:DeprecationWarning', '-c', code]
  tests.run_python(path=None, args=args, expect_pass=True)
```

```sh
python
pip install -U eventlet
pip install -U https://github.com/eventlet/eventlet/archive/master.zip
cd doc
make html
```

```
```


