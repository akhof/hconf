# hconf
hconf is a little module to manage simple configurations I made for myself. Maybe you need this...

# How to use
## Classes
    Hconf
        + __init__(fieldDefinition, path, serialer=Serialer)
        + existPath()                                                     return True if the path exists
        + load()                                                          Loads the hconf
        + save()                                                          Saves the hconf
        + SET(key, value)                                                 Sets the field -key- to the value -value-
        + GET(key)                                                        Returns the value of the field -key-
        + GET_ALL()                                                       Returns a dict with all fields {key:value}
        (+ invalidVersionFound(myVersion, foundVersion, foundValues)      This function is called if while loading the Hconf a version found with an other version-code (normaly this func. im empty. You have to override in an own class)
                                                                              -myVersion- is the excepted version -foundVersion- is the in the Hconf found version -foundValues- are the vaues found in the Hconf
                                                                              if this function returns True the pöd hconf is converted successfull. You have to save the hconf if you change sth. with self.SET()
    FieldDefinition
        + __init__(version=1)
        + addField(field)                                                 adds a field to the Hconf-FileDefinition
    
    Field
        + __init__(key, default=None, types=[])                           the types-list contains all datatypes this field can have. If there's no entry all types are excepted.
    
    Serial
        + _serial(o)                                                      this function has to be overridden. -o- contains a dict which hs to be 'converted' to a string which has to be returned
        + _unserial(o)                                                    this function has to be overridden. -o- contains a 'converted' string which has to be 'unconverted' to a dict which has to be returned
    
    cPickleSerial  <- Serial                                              this class is a serialer which uses cPickle and not pickle. This is much faster if you want to convert big or many data


## Exceptions
    _base_                                                                all exceptions contains the functions of this class
        + getTraceback()
        + getMsg()

    CannotSaveHconf  <- _base_                                            this exception is raised if there was an error while saveing Hconf
    CannotLoadHconf  <- _base_                                            this exception is raised if there was an error while loading Hconf
    InvalidKey       <- _base_, __builtin__.KeyError                      this exception is raised if an invalid key was tried to set or get
                                                                              you can catch this with the built-in KeyError
    InvalidFieldType <- _base_, __builtin__.TypeError                     this exception is raised if you tried to give an field an invalid value
                                                                              you can catch this exception with the built-in TypeError

## Example
def _example():
    import pickle, datetime

    # Here the fields of th Hconf are defined
    class myFieldDefinition(FieldDefinition):
        def __init__(self):
            FieldDefinition.__init__(self, 2)
            self.addField( Field("lang", "en", [str]) ) # the field 'lang' with the default-value 'en' and the type 'str'
            self.addField( Field("lastlogins", [], [list]) )
            self.addField( Field("username", "", [str]) )
            self.addField( Field("password", "", [str]) )
            self.addField( Field("number", 0, [int, float]) ) # the field 'number' can contains the types 'int' or 'float'
            self.addField( Field("alwaysStart", True, [bool, int]) )
            self.addField( Field("dummy") ) # the field 'dummy' has no default-value (=None) and can contains all types

    # Here an own serialer is defined to serial the Hconf (write to file / load from file)
    class mySerialer(Serialer):
        def _serial(self, o):
            return pickle.dumps( o )
        def _unserial(self, o):
            return pickle.loads( o )

    # Here an own Hconf is defined (only to override the function 'invalidVersionFound'; if you don't want to override this function you can create an instance of Hconf)
    class myHconf(Hconf):
        def invalidVersionFound(self, myVersion, foundVersion, foundValues):
            print("found an invalid version (old:{}, my:{}!".format(foundVersion, myVersion))
            # Try to ocnvert the old version th the new version...
            if foundVersion > 2:
                return False # converting was not successfull, because the version of the existing Hconf-file is to new (version > 2) and you don't know how it is built
            else:
                try:
                    # all values of version 2 has the defaut-values
                    # the version 1 has only the fields "lang", "username" and "pwd". You have to know the FieldDefinition of Version 1. This is not directly defined here.
                    self.SET("lang", foundValues["lang"])
                    self.SET("username", foundValues["username"])
                    self.SET("password", foundValues["pwd"])
                    self.save() #!!!
                    return True #return True means that the convertig was successfull. You have to save the Hconf if you made changes!
                except:
                    return False

    
    ### ### ### ### ### ### ### ### ### ### ### ### ### ### ### ### ### ### ### ### ### ### ### ### ### ### ### ### ### ### ### ### ### ### ### ### ### ### ### ### ### ###


    path = "/home/you/username/any/file"
    action = "save" # 'save' or 'load'
    useDefaultSerialer = False

    if useDefaultSerialer:
        # The built-in converter which uses pickle and base64 to serial the Hconf
        h = myHconf(myFieldDefinition(), path)
    else:
        # The serialer you defined above
        h = myHconf(myFieldDefinition(), path, mySerialer)
    
    if action == "save":
        h.SET("lang", "de")
        h.SET("username", "root")
        h.SET("password", "aNotSoGoodPassword")
        h.SET("lastlogins", [datetime.date(2015, 2, 12), datetime.date(2015, 2, 9), datetime.date(2015, 9, 4)])
        h.SET("number", 123.456)
        h.SET("alwaysStart", False)
        h.save()
    else:
        h.load()
        print(h.GET_ALL())
