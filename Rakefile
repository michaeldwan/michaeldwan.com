def banner(message)
  puts "=== #{message} ==="
end

def print(message)
  Kernel.print message
  STDOUT.flush
end

desc "Remove generated files"
task :clean do
  banner "Cleaning"
  `rm -rf build/*`
  `rm -rf .sass-cache/*`
end

task :build do
  banner "Building"
  `bundle exec jekyll build`
end

desc "Deploy to S3"
task :deploy => [:clean, :build] do
  require './env' # env.rb file
  # replace this hot mess with the new aws-sdk-core gem when it's released
  require 'aws-sdk'

  AWS.config({
    access_key_id: ENV["aws_access_key_id"],
    secret_access_key: ENV['aws_secret_access_key'],
    region: ENV['aws_region']
  })

  s3 = AWS::S3.new

  bucket = s3.buckets[ENV['aws_s3_bucket']]

  current_objects = {}

  bucket.objects.each do |object|
    current_objects[object.key] = object
  end

  files = Dir["build/**/*"]

  banner "Uploading #{files.length} files"

  uploaded, skipped = 0, 0

  files.each do |path|
    next if File.directory?(path)
    key = path.gsub(/^build\//, "")

    print "  #{key}"

    obj = current_objects[key]

    if obj && obj.etag.gsub(/"/, '') == Digest::MD5.hexdigest(File.new(path).read)
      skipped += 1
      puts ", skipping"
    else
      obj = bucket.objects[key]
      obj.write(Pathname.new(path))
      obj.acl = :public_read
      uploaded += 1
      puts ", done"
    end

    current_objects.delete(key)
  end

  puts
  puts "  #{uploaded} uploaded, #{skipped} skipped"

  if current_objects.length > 0
      banner "Found #{current_objects.length} stale files on server"
    current_objects.each do |key, object|
      print "  removing #{key}"
      object.delete
      puts ", done"
    end
  end

  banner "Deploy Complete!"
end
