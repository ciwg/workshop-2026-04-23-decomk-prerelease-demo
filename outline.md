* Decomk needs a separate configuration repo that is specific to the group/org/etc  
  * JJ made one called workspace-config, based on  a pre-release version of the decomk requirements for config.    
  * Steve forked that and named the fork decomk-conf-cswg and is converting that for post-release production work.   
  * Major differences between pre-release and post-release decomk config in target repos:  
    * env vars in .devcontainer/devcontainer.json have changed  
      * go install ...@latest does not pull the most recent commit, but instead the one with a version number tagged on it. @stable could also be a choice that would make sense  
        * Steve points out that DevOps releases work very differently from normal application releases, instead of deployment being someone else's problem it is the very problem you are dealing with  
    * no Dockerfile needed or wanted in target repos \-- everything goes in Makefile for single-path  
    * image management is better described now, in decomk/doc/image-management.md \-- configuration repo also acts as image producer, target repos are based on ("consume") those images   
  * Steve is using mob-sandbox as the guinea pig for testing this before it is used to build codespaces for fpga-workbench and other repos   
  * JJ is meanwhile converting fpga-workbench/.devcontainer to the post-release decomk config in parallel.  Specifically:  
    * no dockerfile \-- all build steps go in Makefile  
    * decomk runs during both updateContentCommand (during prebuild) as well as postCreateCommand (during first boot)  
  * event sourcing:  accounting journal  
    * transactions are created from source documents  
  * command sourcing: source documents, in chronological order  
    * reality (state) is created from source documents  
    * building analogy  
    * a makefile stanza is a source document \-- don't edit it  
    * blockchain cryptocurrencies, the blockchain itself, is a command source  
      * a cryptocurrency transaction is a piece of code, more similar to a source document than an accounting transaction.  
      * worth remembering that blockchains do not, cannot, edit history \-- that the entire purpose of the hash chaining is to detect and prevent tampering with history  
      * version 4 of isconf actually uses hash chaining to prevent editing history, but Steve found it to be too constraining for those rare cases when you need to, e.g. a fubar involving the interaction between apt repositories expiring older packages vs local caching of those same packages   
        * apt, as good as it is, is pretty hostile toward the idea of being able to build a machine to the state it was in X years ago \-- you have to do your own capturing and storing of .deb files before the expire off of servers, and you will encounter bugs in your code that manages that  
        * This category of problem is one of the things that, decades ago, led Steve to believe that a different model is needed \-- isconf and decomk are themselves just stopgaps.  
        * the problem has become worse as networked computing increases, and exists because the operating systems themselves were not originally designed at the kernel level to be managed over a network \-- the syscalls and other primitives don't support the sort of journaling (command sourcing), permissioning, and security that is needed for reliable reproduction and testing of off-host origins, etc.
