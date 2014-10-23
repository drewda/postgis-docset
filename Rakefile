require 'sqlite3'
require 'nokogiri'

VERSION = '2.1'
DOC_URL = "http://postgis.net/docs/manual-#{VERSION}/"

task :default => [
  :fetch_docs,
  :index,
  :add_info_plist,
  :copy_icon,
  :tar
]

task :fetch_docs do
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
  info_plist = Dir.getwd + '/Info.plist'
  FileUtils.cp info_plist, docset_contents_path
end

task :copy_icon do
  icon = Dir.getwd + '/icon.png'
  FileUtils.cp icon, docset_contents_path + '..'
end

task :tar do
  system "cd dist && tar --exclude='.DS_Store' -cvzf Postgis.tgz postgis.docset"
end

private

  def docset_contents_path
    Dir.getwd + '/dist/postgis.docset/Contents/'
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
