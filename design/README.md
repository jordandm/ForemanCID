# ForemanCID

## Overview
ForemanCID is a Continuous Integration (CI) and Deployment (CD) server based on a Ruby DSL. Using the Foreman DSL you describe a 'recipe' for building a project.

## Foreman DSL
The Foreman DSL is inspired by, and steals shamelessly, from the Chef DSL. This is intentional and
is done to make it easy for users of Chef to use ForemanCD and extend it.

## Foreman Providers
ForemanCD has the following standard providers which **are not** part of the Chef providers.

### Trigger
The `trigger` provider defines the events which trigger a recipe.  The standard trigger events are *timer*, *repo*, *file*, *directory*, *message*, and *web_hook*.

#### Timer Trigger
Timer triggers allow you to trigger an action on a periodic basis. 

    # Each time a cron alarm occurs, we will create an event message. If handle is :last, when
    # adding a new event message to the queue, we will remove the last one added; if handle is 
    # :all then all event messages will be added to the queue.
    trigger 'It is 0530 PST' do
      when timer do
        frequency ['30 5 * * *'] # Standard cron descriptions, can have multiple 
        handle :last # :all or :last; do we only handle the last cron alarm?
      end 
      action :create
      notify {:service_1 => :message, :sevice_2 => :message}
    end

#### Repo Trigger
Repo triggers allow you to trigger an action when a repo changes. You need to specify the type of repo (git, etc.) in the `when` clause. 

Initially, we will only be supporting git, but the Repo class can be subclassed and extended to support other repo types.

    # If we are repo check is a poll, then we will get a list of changes committed since our last
    # check and either create a message for each one or only for the last one made.
    trigger 'foremancd head of master updated' do
      when repo_git do
        name 'foremancid'
        remote 'mygitrepo.domain.com'
        protocol :ssh
        branch 'master' # RegEx describing the branches to be monitored
        key foreman_cd_private_key # Private Key to access the repo
        type :poll # :poll or :commit; do we poll the repo or do we respond on a commit trigger
                   # If we are using a commit trigger, then the end point is
                   #    foreman_host/commits/repo/branch 
                   # the package includes the various details
        frequency ['*/2 * * * *'] # If provided for type :commit, ignored
        handle :all # :all or :last; do we only handle the last commit or all commits?
      end 
      action :create
      notify {:service_1 => :message, :sevice_2 => :message}
    end

#### Directory Trigger
Directory triggers allow you to trigger an action when a directory changes. YOu can filter changes on a regex.

    # It is possible that directory changes will be unobserved, if they occur within the frequency
    # of directory change checks on the same file in the directory.
    trigger 'directory /foremancd/artifacts/foreman_builds/ update' do
      when directory do
        path '/foremancd/artifacts/foreman_builds'
        frequency ['*/5 * * * *']
        context \RegEx\
        handle :all # :all or :last, do we trigger a run for all changes or only the last?
      end
      action :create
      notify {:service_1 => :message, :sevice_2 => :message}
    end


##### File Trigger
File triggers allow you to trigger an action when a file changes. You can filter changes based on a regex.

    # It is possible that file changes will be unobserved, if they occur within the frequency
    # of file change checks.
    trigger 'file /foremancd/artifacts/foreman_builds/build_list update' do
      when directory do
        file '/foremancd/artifacts/foreman_builds/build_list update'
        frequency ['*10 * * * *'] # How frequently do we check for changes?
        content \RegEx\
        handle :last # :all or :last, do we trigger a run for all changes we observed or only the last?
      end
      action :create
      notify {:service_1 => :message, :sevice_2 => :message}
    end

##### Message Trigger
Message triggers allow you to trigger an action by sending a message using the AMQP protocol. Queues of messages are maintained, which you can handle in the order you decide.

    # If handle is :last, we remove any messags in the queue when insert a new one. If handle is
    # :all, then we just insert the message in the queue.
    trigger 'message new_stuff_for_me' do
      when message do
        queue 'new_stuff_for_me'
        key 'some_key'
        order :fifo # :fifo, :lifo, :key is our queue FIFO or LIFO or Key based?
        handle :last # :all or :last, do we handle all messages in queue or just the last?
      end
      action :create
      notify {:service_1 => :message, :sevice_2 => :message}
    end

Each project has an event queue. When a trigger event is observed by ForemanCD 

### Perquisite
The `perquisite` provider defines a resources which is required to run a recipe.

### Consume
The `consume` provider allows a recipe to consume event that were published elsewhere.

### Produce
The `produce` provider allows a recipe to publish or produce events that will be handle elsewhere.

### Build
The `build` provider defines a build step to be run in a recipe.

### Package
The `package` provider defines a packaging process to be used by a recipe. 








