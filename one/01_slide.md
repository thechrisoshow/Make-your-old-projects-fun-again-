!SLIDE 
# Refactoring for fun and profit #

!SLIDE
# Chris and Tim
# @thechrisoshow @misterim


!SLIDE bullets incremental
# Programming is fun!
* Until it's not
* Let's make it fun again!

!SLIDE 
# One teams war story #

!SLIDE 
# Creativenvironment #

!SLIDE 
# The Scheduler #
# (aka The Black Hole of Doom)


!SLIDE small
    @@@ ruby
    def schedule_task(task, args) 
      if task.should_be_forced_into_bucket?
        forced_schedule_task(task) 
      elsif args[:immediate] == true
        schedule_task_immediately(task, args)
      else
        if args[:on]
          task.fill_options ||= {}
          task.fill_options[:event_day] = 
                      args[:on]
          task.save if task.changed?
        end

        schedule_date = args[:on] || 
                          Date.today
        scheduling = schedule_task_as_best_fit(
          task, schedule_date, args[:base_time])
  
        return scheduling
      end
    end


!SLIDE 
# What did we need to change, how did we change it? #

!SLIDE
# User stories #

!SLIDE
# Cucumber features #

!SLIDE bullets incremental
# envjs is amazing

* But the capybara plugin for it isn't quite ready for prime time

!SLIDE
# Capybara's support for cucumber freaking rocks

!SLIDE
# How to turn on Selenium Testing

    @selenium <-- BOOM
    Scenario: Setting a job rate
      When I fill in "Job price" with "100.00"
      And I choose "Update all tasks"
      And I press "Save"

      Then I should see "Job rate was 
        successfully updated"


!SLIDE
# Pickle
    Given a company: "HPK" exists

    And a user "Bob" exists with email: 
      "bob@example.com", company: company "HPK", 
      state: "active", admin: true

    And a job "Speculative job" exists with 
      company: company "HPK", owner: user "Bob", 
      name: "Speculative job"    
      
      
    Then task: "Existing task" should exist with 
        role_price_in_cents: 10000
   
!SLIDE
# Timecop and Chronic
    @@@ruby
    
    # Given the date is 21 June 2010
    Given /^the (date|time) is (.+)$/ do |_,s|
      Timecop.freeze(Chronic.parse(s))
    end

    # Given it's 1 week in the future
    Given /^it's (.+) in the future$/ do |s|
      Timecop.freeze(Chronic.parse("in #{s}"))
    end

!SLIDE
# Planning the refactoring #

!SLIDE
# The actual coding #

!SLIDE small
# Big ole migration #

    @@@ruby
    #Create completions for all tasks marked as complete
    execute <<-SQL
      INSERT INTO completions (task_id, created_at, 
        updated_at)
      SELECT id as task_id, completed_at as created_at, 
        NOW() as updated_at from tasks
      WHERE completed_at IS NOT NULL
    SQL


!SLIDE small
# With assertions #
    @@@ruby
    assert("All users have a corresponding bucket") do
      lambda { 
        count("SELECT count(id) FROM users")
      }.equal {
        count("SELECT count(id) FROM buckets where
         owner_type = 'User'")
      }
    end

!SLIDE bullets incremental
# cap production sync:db

* Kinda like taps in heroku

!SLIDE
# Ensure the behaviour stays the same

!SLIDE small
# Wrote very targeted specs
    @@@ruby
    before(:each) do
     Timecop.freeze(Date.parse("2010/07/27"))
     Estimate.any_instance.stubs(:duration).returns(8.hours)
     @bucket = Bucket.make
     @entry = FloatingEntry.make(:position => 5, 
        :start_on => Date.yesterday, :bucket => @bucket)
    end

    it "does not take up time before the start date" do
      @entry.time_on(date("Last friday")).should == 0
    end

!SLIDE
# Having excellent cucumber features made the refactoring non-mental #

!SLIDE
# We're always on the look out for smart people to work with #

