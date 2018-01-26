require 'faraday'
require 'json'
require 'json-pointer'
require 'yaml'

task default: [:download, :rewrite]

task :download do
  response = Faraday.new('http://webconcepts.info').get('/specs.json')
  raise "Cannot download webconcepts/specs.json" unless response.success?
  File.write('tmp/specs.json', response.body)
end

task :rewrite do

  interest = YAML.load(File.read('./_data/interest.yml'))
  specs_data = JSON.parse(File.read('./tmp/specs.json'))

  # Grab them specs from specs.json
  relevant = interest['webconcepts'].flat_map do |body_key, specs|
    body = specs_data[body_key]
    specs.map do |spec|
      pointer = JsonPointer.new(body, spec['pointer'])
      raise "Pointer #{spec['pointer']} does not exist" unless pointer.exists?
      pointer.value.tap do |v|
        v['body'] = {
          'URL' => body['id'],
          'short' => body['short']
        }
      end
    end
  end

  relevant += interest['custom']

  relevant.sort_by do |spec|
    spec['title'] unless spec['title'].nil?
  end

  File.write('_data/standards.yml', YAML.dump(relevant))
end
