def banner(message)
  puts "=== #{message} ==="
end

def print(message)
  Kernel.print message
  STDOUT.flush
end

def jekyll_config_files
  ["_config.yml", "_local_config.yml"].select { |f| File.exist?(f) }.join(',')
end

desc "Serve the Jekyll site"
task :serve do
  system("bundle exec jekyll serve --watch --config #{jekyll_config_files}")
end

desc "Remove generated files"
task :clean do
  banner "Cleaning"
  `rm -rf build/*`
  `rm -rf .sass-cache/*`
end

task :build do
  banner "Building"

  system("bundle exec jekyll build --config #{jekyll_config_files}")
end

desc "Deploy to S3"
task :deploy => [:clean, :build] do
  require 'dotenv'
  Dotenv.load
  # replace this hot mess with the new aws-sdk-core gem when it's released
  require 'aws-sdk'

  AWS.config({
    access_key_id: ENV["AWS_ACCESS_KEY_ID"],
    secret_access_key: ENV['AWS_SECRET_ACCESS_KEY'],
    region: ENV['AWS_REGION']
  })

  s3 = AWS::S3.new

  bucket = s3.buckets[ENV['AWS_S3_BUCKET']]

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
