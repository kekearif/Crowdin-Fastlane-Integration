Simply add this lane to the `Fastfile`, make sure `ProjectName` is replaced with the project name and then configure and add the `Crowdinfile.yml` to the `fastlane` dir in your project.

```ruby

  # This currently only updates Localizable.strings
  # Regions.strings and InfoPlist.strings must be done manually
  desc "Update the string files from Crowdin"
  lane :strings do
    UI.user_error!("Crowdinfile.yml is required to run this lane! Aborted \u{1F6D1}") unless File.exist?('Crowdinfile.yml')
    config = YAML.load(File.read("Crowdinfile.yml"))
    build = "https://api.crowdin.com/api/project/#{config["project_id"]}/export?key=#{config["api_key"]}"
    dl = "https://api.crowdin.com/api/project/#{config["project_id"]}/download/all.zip?key=#{config["api_key"]}"
    path = File.expand_path('..') + "/ProjectName/Strings/"
    langs = config["langs"]
    # Expected size of the array, used to check if new languages have been added or not
    # The size logic can be omitted if needed
    size = 52

    response = Net::HTTP.get_response(URI(build))
    if response.kind_of? Net::HTTPSuccess
      puts "Strings project built! \u{1F57A}"
      puts "Downloading zip file...."
      content = open(dl)
      puts "Updating files...."
      Zip::File.open_buffer(content) do |zip|
        # The size logic can be omitted if needed
        UI.user_error!("Additional languages have been added, please reconfigure yml file!! \u{1F6D1}") unless zip.entries.count == size
        zip.each do |entry|
          if langs.key?(entry.name)
            # Key found, pull content and write to file
            langs[entry.name].each do |lang|
              content = entry.get_input_stream.read
              full_path = path + lang + "/Localizable.strings"
              puts "Updated! \u{1F4AA} " + full_path
              File.open(full_path, 'w') { |file| file.write(content) }
            end
          end
        end
      puts "String update complete! \u{1F984}"
    end
    else
      UI.user_error!("Failed to build project strings \u{1F622}")
    end
  end
  ```
