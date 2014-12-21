Contract verification in Python
###############################
.. author:: Shekhar
.. tags:: python, programming, zope, zinterfaces

.. code-block:: python

    import zope.interface.verify
    
    class ITest(zope.interface.Interface):
       def foo(arg1): pass
       def bar(): pass   
    
    class Test(object):
       zope.interface.implements(ITest)
       def foo(self): pass
    
    class Test2(object):
       zope.interface.implements(ITest)
       def foo(self, arg1): pass
      
    class Test3(object):
       zope.interface.implements(ITest)
       def foo(self, arg1): pass
       def bar(self): pass
    
    for cls in (Test, Test2, Test3):
       try:
           if zope.interface.verify.verifyClass(ITest, cls):
               print "OK: %s correctly implements %s" % (cls.__name__, ITest.__name__)
       except Exception, err:
           print "Error detected with %s's implementation: %s" % (cls.__name__, err)
