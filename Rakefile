require 'yaml'
require 'wmq'
require 'json'
require 'active_support/core_ext/hash/conversions'
require 'pp'

desc "The default task is to connect and fetch all messages of the default type test."
task :default => ["connect_and_fetch"]


desc "Set up a connection and fetch messages for the given message_type"
task :connect_and_fetch, [:message_type] do |t, args|
  args.with_defaults(:message_type => "my_message_type", :second_arg => "Bar")

  message_type = args.message_type
  puts "Message type was: #{message_type}"

  config = get_config(message_type)

  puts "Connecting to #{config['message_channel']['host_ip']}(#{config['message_channel']['port']})..."
  puts "#{config['message_channel']['host_name']}(#{config['message_channel']['port']})"
  puts "q_mgr_name: #{config['message_channel']['q_mgr_name']}"
  puts "q_name: #{config['message_channel']['q_name']}"
  puts "channel_name: #{config['message_channel']['channel_name']}"
  puts "user_identifier: #{config['message_channel']['user_identifier']}"
  puts "exception_on_error: #{config['message_channel']['exception_on_error']}"
  puts "trace_level: #{config['message_channel']['trace_level']}"

  ## Set up the connection here
  WMQ::QueueManager.connect(
    connection_name: "#{config['message_channel']['host_name']}(#{config['message_channel']['port']})",
    #q_mgr_name: config['message_channel']['q_mgr_name'],
    q_name: config['message_channel']['q_name'],
    channel_name: config['message_channel']['channel_name'],
    user_identifier: config['message_channel']['user_identifier'],
    exception_on_error: config['message_channel']['exception_on_error'],
    trace_level: config['message_channel']['trace_level'],
  ) do |qmgr|
    qmgr.open_queue(q_name: "#{config['message_channel']['q_name']}", mode: :input) do |queue|

      message = WMQ::Message.new
      if queue.get(message: message)

        puts '================'
        #puts "Data Received: #{message.data}"
        puts '================'
        puts 'Message Descriptor:'
        puts "msg_id: #{message.descriptor[:msg_id]}"
        puts "put_date: #{message.descriptor[:put_date]}"
        puts "put_time: #{message.descriptor[:put_time]}"
        puts "msg_type: #{message.descriptor[:msg_type]}"
        puts "coded_char_set_id: #{message.descriptor[:coded_char_set_id]}"
        puts '================'
        puts 'Headers Received:'
        message.headers.each { |header| p header }
        puts '================'
        puts '----------------'

        put_date = "#{message.descriptor[:put_date]}-#{message.descriptor[:put_time]}"
        
        fetch_date = Time.now.strftime('%Y%m%d-%H%M%S')

        descriptor = message.descriptor

        print_file(message_type, "#{fetch_date}_DATA", put_date, message.data, "xml")
        print_file(message_type, "#{fetch_date}_DESC", put_date, descriptor, "txt")

        # fix encoding of message_data
        reencoded_data = reencode_data(message.data, 'ISO-8859-1', 'utf-8')
        
        # print to json file or forward json message
        json_data = convert_to_hash(reencoded_data).to_json
        
        print_file(message_type, "#{fetch_date}_APP", '', json_data, "json")
      else
        puts 'No message available'
      end
    end
  end
end

desc "convert to hash."
task :convert_to_hash, [:file_name] do |t, args|
  args.with_defaults(:file_name => "my_example_file.xml")
  file_name = args.file_name
  data = get_data_from_file(file_name, 'ISO-8859-1', 'utf-8')
  hash_data = convert_to_hash(data)
  json_data = hash_data.to_json
  pp json_data
end

def convert_to_hash(data)
  start = data.index("<?xml")
  stop = data.length
  new_data = data.slice(start..stop)
  hash_data = Hash.from_xml new_data
  return hash_data
end

def reencode_data(data, source_encoding, target_encoding)
  data.force_encoding('ISO-8859-1').encode('utf-8')
end

def get_data_from_file(file_name, source_encoding, target_encoding)
  File.open(file_name, "rb") do |f|
    data = f.read
    f.close
    return reencode_data(data, source_encoding, target_encoding)
  end
end

def print_file(message_type, some_kind_of_id, message_date, data, file_extension)
  File.open("log.#{message_type}_#{some_kind_of_id}_#{message_date}.#{file_extension}", "wb") do |f|
    f.write(data)
    f.close
  end
end

def get_config(message_type)
  yaml = YAML.load_file('config.yml')
  yaml['config']['message_types'][message_type]
end

def reencode_array(array, encoding)
  array.map do |item|
    if item.kind_of?(Hash)
      item = reencode_hash(item, encoding)
    elsif item.kind_of?(Array)
      item = reencode_array(item, encoding)
    elsif item.kind_of?(String)
      item = item.dup.force_encoding(encoding)
    end
    item
  end
end

def reencode_hash(hash, encoding)
  new_hash = {}
  hash.each_pair do |key,val|
    if val.kind_of?(Hash)
      new_hash[key] = reencode_hash(val, encoding)
    elsif val.kind_of?(Array)
      new_hash[key] = reencode_array(val, encoding)
    elsif val.kind_of?(String)
      new_hash[key] = val.dup.force_encoding(encoding)
    else
      new_hash[key] = val
    end
  end
  new_hash
end
