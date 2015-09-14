GO_PATH = File.join(Dir.home, 'Documents', 'projects', 'go')
LOXIGEN_DIR = File.join(Dir.home, 'Documents', 'projects', 'loxigen')

LOXIGEN_GO_ARTEFACTS = Dir.glob(File.join(LOXIGEN_DIR, 'loxi_output', 'goloxi', 'loxi', 'of10','*.go'))

ELASTICSEARCH_HOST = 'localhost'
ELASTICSEARCH_PORT = 9200

PACKETBEAT_DIR = File.join(GO_PATH, 'src', 'github.com', 'elastic', 'packetbeat')
PACKETBEAT_OPENFLOW_DIR = File.join(PACKETBEAT_DIR, 'protos', 'openflow')
PACKETBEAT_TEST_DIR = File.join(PACKETBEAT_DIR, 'tests')
PACKETBEAT_INDICES = '/packetbest-*/'

TRACE_FILE_NAME = 'openflow.pcap'
CONFIG_FILE_NAME = 'packetbeat.yml'
OUTPUT_DIR = 'output'
OUTPUT_PATTERN = File.join(OUTPUT_DIR, 'packetbeat*')
OUTPUT_FILES = Dir.glob(File.join(OUTPUT_DIR, OUTPUT_PATTERN))

task :build_loxigen do
  cd LOXIGEN_DIR do
    sh 'make go'
  end
end

task copy_loxigen_artifacts: :build_loxigen do
  cp LOXIGEN_GO_ARTEFACTS, PACKETBEAT_OPENFLOW_DIR
end

task generate_stringer: :copy_loxigen_artifacts do
  cd PACKETBEAT_OPENFLOW_DIR do
    sh 'go generate'
  end
end

task build_packetbeat: :copy_loxigen_artifacts do
  cd PACKETBEAT_DIR do
    sh 'make'
  end
end

task :clean_packetbeat_output do
  rm OUTPUT_FILES
end

task :clean_elasticsearch do
  require 'net/http'
  Net::HTTP.new(ELASTICSEARCH_HOST, ELASTICSEARCH_PORT).delete(PACKETBEAT_INDICES)
end

task packetbeat_unit_tests: :build_packetbeat do
  cd PACKETBEAT_DIR do
    sh 'make test'
  end
end

task process_demo_trace: [:clean_packetbeat_output, :clean_elasticsearch, :build_packetbeat] do
  trace_file = File.join(Dir.pwd, 'traces', TRACE_FILE_NAME)
  packetbeat_config_name = File.join(Dir.pwd, 'config', 'packetbeat.yml')
  packetbeat_bin = File.join(PACKETBEAT_DIR, 'packetbeat')
  sh "#{ packetbeat_bin } -v -e -I #{ trace_file } -c #{ packetbeat_config_name }"
end

task :clean do
  rm_rf 'output'
  rm_rf 'elasticsearch'
  rm_rf 'kibana'
end

task install: :clean do
  mkdir 'output'

  get_and_rename '1.7.1', 'elasticsearch'  do |version|
    "elasticsearch-#{ version }.zip"
  end

  get_and_rename '4.1.2', 'kibana'  do |version|
    "kibana-#{ version }-darwin-x64.zip"
  end
end

def get_and_rename(version, product)
  filename = yield version
  url = "https://download.elastic.co/#{ product }/#{ product }/#{ filename }"
  `wget #{ url }`
  `unzip #{ filename }`
  rm filename
  mv filename.gsub(".zip", ""), product
end

task default: [:packetbeat_unit_tests, :process_demo_trace]
