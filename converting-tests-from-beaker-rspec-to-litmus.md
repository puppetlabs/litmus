# Introduction
Below we will list some example patterns you may want to use in your tests, when moving from beaker-rspec style testing. 

## where to put your helper functions
generally helper functions used to live everywhere, special files, in the spec_helper_acceptance.rb

     spec\
     spec\spec_helper_acceptance.rb        <-some functions in here
     spec\acceptance\helper_functions.rb   <-some functions in here
     spec\acceptance\test_spec.rb          <-some functions in here

**new way** we put all helper code in one place, it will be automatically loaded by spec_helper_acceptance.rb

     spec\spec_helper_acceptance_local.rb   <- all helper code should live in here

## a basic test example

Below is a standard trope for checking that your puppet code works, it is a repeatable pattern.

    require 'spec_helper_acceptance'

    describe 'concat noop parameter', if: ['debian', 'redhat', 'ubuntu'].include?(os[:family]) do
      before(:all) do
        @basedir = setup_test_directory
      end
      describe 'with "/usr/bin/test -e %"' do
        let(:pp) do
          <<-MANIFEST
          concat_file { '#{@basedir}/file':
            noop => false,
          }
          concat_fragment { 'content':
            target  => '#{@basedir}/file',
            content => 'content',
          }
        MANIFEST
        end

        it 'applies the manifest twice with no stderr' do
          idempotent_apply(pp)
          expect(file("#{@basedir}/file")).to be_file
          expect(file("#{@basedir}/file").content).to contain 'content'
        end
      end
    end

## checking your manifest code is idempotent
old way you would apply the manifest twice checking for different things.

    pp = ' class { 'mysql::server' } '
    execute_manifest(pp, catch_failures: true)
    execute_manifest(pp, catch_changes: true)

new way with litmus, we can use the idempotent_apply helper function. (its quicker too) 

    pp = ' class { 'mysql::server' } '
    idempotent_apply(pp)

## running shell commands

the shell command has become run_shell. Generally in the past code blocks were used.

     shell('/usr/local/sbin/mysqlbackup.sh') do |r|
       expect(r.stderr).to eq('')
     end

This can be done on a single line, if you are only checking one thing from the command

    expect(run_shell('/usr/local/sbin/mysqlbackup.sh').stderr).to eq('')

## checking facts
Calling facter or getting other system information was like:

    fact_on(host, 'osfamily')
    fact('selinux')

You can now use the serverspec functions (incidentally, these are cached so are quick to call) look here for more https://serverspec.org/host_inventory.html 

    os[:family]
    host_inventory['facter']['os']['release']