require 'sqlite3'
require 'nokogiri'

VERSION_IN_URL = 'dev'
VERSION_IN_FILENAME = '2.3.0dev'
DOC_URL = "http://postgis.net/docs/manual-#{VERSION_IN_URL}/"

task :default => [
  :fetch_docs,
  :index,
  :add_info_plist,
  :copy_icon,
  :tar
]

task :fetch_docs do
  Dir.mkdir docset_root_path
  target_dir = docset_contents_path + '/Resources/Documents'
  system "wget --convert-links --page-requisites --recursive --no-parent --cut-dirs=2 -nH --directory-prefix=#{target_dir} #{DOC_URL}"
end

task :index do
  db_path = docset_contents_path + '/Resources/docSet.dsidx'
  db = SQLite3::Database.new db_path
  create_docset_table(db)
  parse_doc_into_db(db)
end

task :add_info_plist do
  info_plist = Dir.getwd.chomp('/') + '/Info.plist'
  plist_contents = File.read(info_plist).gsub('{VERSION_IN_FILENAME}', VERSION_IN_FILENAME);
  File.open(docset_contents_path + '/Info.plist', 'w') { |file| file.write(plist_contents) }
end

task :copy_icon do
  icon = Dir.getwd.chomp('/') + '/icon.png'
  FileUtils.cp icon, docset_contents_path + '/../'
end

task :tar do
  Dir.chdir "dist"
  system "7z.exe a -ttar postgis-" + VERSION_IN_FILENAME + ".tar postgis-" + VERSION_IN_FILENAME + ".docset"
  system "7z.exe a postgis-" + VERSION_IN_FILENAME + ".tgz postgis-" + VERSION_IN_FILENAME + ".tar"
  File.delete("postgis-" + VERSION_IN_FILENAME + ".tar")
  Dir.chdir ".."
end

private

	def docset_root_path 
     Dir.getwd.chomp('/') + '/dist/postgis-' + VERSION_IN_FILENAME + '.docset'
	end

  def docset_contents_path
     docset_root_path + '/Contents'
  end

  def create_docset_table(db)
    db.execute <<-SQL
      CREATE TABLE IF NOT EXISTS searchIndex(id INTEGER PRIMARY KEY, name TEXT, type TEXT, path TEXT);
      DELETE FROM searchIndex;
      CREATE UNIQUE INDEX anchor ON searchIndex (name, type, path);
    SQL
  end

  def doc(file_name)
    file = File.open(docset_contents_path + '/Resources/Documents/' + file_name)
    doc = Nokogiri::HTML(file)
    file.close
    doc
  end

  def parse_doc_into_db(db)
    # chapters
    doc('index.html').css('.chapter').each do |chapter|
      name = chapter.text
      path = chapter.css('a')[0]['href']
      db.execute <<-SQL
        INSERT OR IGNORE INTO searchIndex(name, type, path) VALUES ('#{name}', 'Guide', '#{path}');
      SQL
    end

    # functions
    # TODO: look for functions outside of the ST_*, RT_*, TP_* files
    all_files = Dir.entries(docset_contents_path + '/Resources/Documents')
    function_files = all_files.select { |file_name| file_name =~ /^[A-Z]{2}_/}
    function_files.each do |file_name|
      name = doc(file_name).css('title').text
      db.execute <<-SQL
        INSERT OR IGNORE INTO searchIndex(name, type, path) VALUES ('#{name}', 'Function', '#{file_name}');
      SQL
    end
  end
