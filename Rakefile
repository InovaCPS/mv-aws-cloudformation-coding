#
# vim:ft=ruby
#
# To remember: Ruby/Rake shell commands
#    Ruby
#       %x{ cmd } or `cmd`  - executes 'cmd' and gives cmd OUTPUT
#       system 'cmd'        - executes 'cmd' and gives OS level return code.
#       exec 'cmd'          - executes 'cmd' in the current process and finalizes it.
#    Rake
#       sh 'cmd'            - prints 'cmd' before executing it and gives cmd OUTPUT#       sh 'cmd'            - prints 'cmd' before executing it and gives cmd OUTPUT
#       sh 'cmd' do |return,output| - executes 'cmd' and gets its OUTPUT and OS return code.
#
# To remeber (2): string quotes
#   '', %q{}, %q[], %q//, %q|| - single quotes
#   "", %Q{}, %Q[], %Q//, %Q|| - double quotes
#   "", %{} , %[] , %// , %||  - double quotes
#
# To remeber (3): heredocs
#   # heredoc, idented, interpolation
#   sh <<-CMD
#       cmd
#       cmd #{var}
#   CMD
#
#   # heredoc, idented, interpolation, remove identation
#   sh <<-'CMD'.gsub(/^[ ]*/,'') -
#       cmd
#       cmd #{var}
#   CMD
#
#   # heredoc, idented, no interpolation (as single quotes)
#   sh <<-'CMD'
#       cmd
#       cmd #{var}
#   CMD
#
# To remeber (4): regex
#   # remove identation, i.e., space at beginning
#   str.gsub(/^[ ]*/,'')
#
#   # remove duplicated space
#   str.gsub(/[ ]+/,' ')
#

##
## Tasks
##
desc "cloudformation: validate-template"
task :'validate-template' => :vt do
end

desc "cloudformation: validate-template"
task :vt do

  begin
    sh %Q{
      cd #{@dir} &&
      aws cloudformation validate-template  \
        --template-body file://#{@file}     \
        --output text                       \
      | tail -n +2 | tr '\\t' ';'           \
      | awk -F';' 'BEGIN {IFS=";"; OFS=":| "} {print $1,$5,$4,$2,$3,$6}' \
      | sort -k2 | column -t -s ':' # order and align parameter list
    }.gsub(/^[ ]*/,'').gsub(/[ ]+/,' ')
  rescue
    printf "\nrake: aws cloudformation: template not valid.\n\n"
  end

end

desc "cloudformation: create-stack"
task :'create-stack' => :cs do
end

desc "cloudformation: create-stack"
task :cs do
  begin
    sh %Q{
      cd #{@dir} &&
      aws cloudformation create-stack     \
          --stack-name #{@name}           \
          --template-body file://#{@file} \
          #{@param} --output json \
    }.gsub(/^[ ]*/,'').gsub(/[ ]+/,' ')
  rescue
    printf "\nrake: aws cloudformation: not created.\n\n"
  end
end

desc "cloudformation: update-stack"
task :'update-stack' => :us do
end

desc "cloudformation: update-stack"
task :us do

  begin
    sh %Q{
      cd #{@dir} &&
      aws cloudformation update-stack     \
          --stack-name #{@name}           \
          --template-body file://#{@file} \
          #{@param} --output json        \
    }.gsub(/^[ ]*/,'').gsub(/[ ]+/,' ')
  rescue
    printf "\nrake: aws cloudformation: no updates.\n\n"
  end

end

desc "cloudformation: delete-stack"
task :'delete-stack' => :ds do
end

desc "cloudformation: delete-stack"
task :ds do

  begin
    sh %Q{
      cd #{@dir} &&
      aws cloudformation delete-stack \
          --stack-name #{@name} \
          --output json
    }.gsub(/^[ ]*/,'').gsub(/[ ]+/,' ')
  rescue
    printf "\nrake: aws cloudformation: not deleted.\n\n"
  end

end


###
### Helpers
###
def usage()
  puts <<-USAGE

Usage:
  rake cs file=<file-name> name=<stack-name> [param=<param-file] [region=aws-region]
  rake us file=<file-name> name=<stack-name> [param=<param-file] [region=aws-region]
  rake vt file=<file-name>

Or:
  file=<file-name> name=<stack-name> [param=<param-file] rake cs
  file=<file-name> name=<stack-name> [param=<param-file] rake us
  file=<file-name> name=<stack-name> [param=<param-file] rake vt

Where:
  file-name:   AWS Cloudformation template file to be used.
  stack-name:  AWS Cloudformation stack name to be created.
               It will use <file-name> if not defined.
  param-file:  Optional: Parameter file in JSON format.
  region:      Optional: AWS region to create the stack.

Parameter file JSON format:
  [
    { "ParameterKey": "MyName"       , "ParameterValue": "MyValue"  },
    { "ParameterKey": "KeyPairName"  , "ParameterValue": "TestKey"  },
    { "ParameterKey": "InstanceType" , "ParameterValue": "t2.micro" }
  ]

  USAGE
end

task :default do
  usage()

  puts "List of tasks:"
  tasks = `rake --tasks`
  tasks.each_line { |line|  puts "  #{line}" }
  puts

end


###
### Settings
###

## caller directory
@dir = ENV['PWD']

## command line parameters
if ENV['file']
  @file = ENV['file']
else
  printf "\nError:\n  file=''\n  Supply a template file for your Cloudformation stack.\n"
  usage()
  exit 1
end

@name  = ENV['name'] || `echo ${file%%.*}`

# TODO: command-line can be param=file.json OR params=file.json
@param = " --parameters file://#{ ENV['param']  } " if ENV['param']
@param = " --parameters file://#{ ENV['params'] } " if ENV['params']

## Hack: valid only inside the script.
ENV['AWS_DEFAULT_REGION'] = ENV['region'] if ENV['region']



