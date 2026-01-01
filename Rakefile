require 'json'
require 'fileutils'
require 'erb'
require 'kramdown'
require 'kramdown-parser-gfm'

def load_config
  config_file = File.expand_path('config.yaml', __dir__)
  config = { 'per_page' => 20 }

  if File.exist?(config_file)
    File.read(config_file).each_line do |line|
      next if line.strip.empty? || line.strip.start_with?('#')
      if line =~ /^\s*(\w+):\s*(.+)\s*$/
        key = $1.strip
        value = $2.strip
        config[key] = value.match?(/^\d+$/) ? value.to_i : value
      end
    end
  end

  config
end

def parse_front_matter(content)
  front_matter = {}
  content_without_fm = content

  if content =~ /\A---\s*\n(?<fm>.*?)\n^---\s*$\n?/m
    fm_block = Regexp.last_match[:fm]
    fm_block.each_line do |line|
      if line =~ /^\s*(\w+):\s*(.*)\s*$/
        key = $1.strip
        value = $2.strip
        front_matter[key] = value
      end
    end
    content_without_fm = content.sub(/\A---\s*\n(?<fm>.*?)\n^---\s*$\n?/m, '')
  end

  [front_matter, content_without_fm]
end

def markdown_to_html(markdown)
  Kramdown::Document.new(markdown, input: 'GFM', syntax_highlighter: nil).to_html
end

def article_url_from_filename(filename)
  # Parse: yyyy-mm-dd-slug.md -> /yyyy/mm/dd/slug.html
  basename = File.basename(filename, ".md")
  if basename =~ /^(\d{4})-(\d{2})-(\d{2})-(.+)$/
    year, month, day, slug = $1, $2, $3, $4
    "/#{year}/#{month}/#{day}/#{slug}.html"
  else
    "/articles/#{basename}.html"
  end
end

def article_output_path(filename, dist_dir)
  # Parse: yyyy-mm-dd-slug.md -> dist/yyyy/mm/dd/slug.html
  basename = File.basename(filename, ".md")
  if basename =~ /^(\d{4})-(\d{2})-(\d{2})-(.+)$/
    year, month, day, slug = $1, $2, $3, $4
    File.join(dist_dir, year, month, day, "#{slug}.html")
  else
    File.join(dist_dir, 'articles', "#{basename}.html")
  end
end

def load_all_articles(articles_dir)
  articles = []

  Dir.glob(File.join(articles_dir, "*.md")).each do |md_file_path|
    content = File.read(md_file_path)
    front_matter, _ = parse_front_matter(content)

    articles << {
      file: md_file_path,
      title: front_matter['title'] || 'Untitled',
      date: front_matter['date'] || Time.now.strftime('%Y-%m-%d'),
      url: article_url_from_filename(md_file_path)
    }
  end

  # Sort by date (newest first)
  articles.sort_by { |a| a[:date] }.reverse
end

namespace :buddie do
  desc "Run server"
  task :server => :build do
    puts "Starting server..."
    FileUtils.cd(File.expand_path('dist', __dir__)) do
      system("ruby -r un -e httpd . -p 8000")
    end
  end

  desc "Generate article HTML pages"
  task :generate_articles do
    config = load_config
    articles_dir = File.expand_path('articles', __dir__)
    dist_dir = File.expand_path('dist', __dir__)
    template_file = File.expand_path('templates/article.html.erb', __dir__)

    unless File.exist?(template_file)
      puts "Warning: Template file #{template_file} not found."
      next
    end

    template = ERB.new(File.read(template_file))

    Dir.glob(File.join(articles_dir, "*.md")).each do |md_file_path|
      content = File.read(md_file_path)
      front_matter, markdown_content = parse_front_matter(content)

      article_title = front_matter['title'] || 'Untitled'
      article_date = front_matter['date'] || Time.now.strftime('%Y-%m-%d')
      article_content = markdown_to_html(markdown_content)

      output_path = article_output_path(md_file_path, dist_dir)
      FileUtils.mkdir_p(File.dirname(output_path))

      html = template.result(binding)
      File.write(output_path, html)
      puts "Generated #{output_path}"
    end
  end

  desc "Generate index pages with pagination"
  task :generate_index_pages do
    config = load_config
    per_page = config['per_page'] || 20
    articles_dir = File.expand_path('articles', __dir__)
    dist_dir = File.expand_path('dist', __dir__)
    template_file = File.expand_path('templates/index.html.erb', __dir__)

    unless File.exist?(template_file)
      puts "Warning: Template file #{template_file} not found."
      next
    end

    template = ERB.new(File.read(template_file))

    all_articles = load_all_articles(articles_dir)
    total_articles = all_articles.length
    total_pages = (total_articles.to_f / per_page).ceil
    total_pages = 1 if total_pages == 0

    puts "Generating #{total_pages} index page(s) (#{total_articles} total articles)"

    (1..total_pages).each do |page_num|
      start_idx = (page_num - 1) * per_page
      end_idx = [start_idx + per_page - 1, total_articles - 1].min
      articles = all_articles[start_idx..end_idx] || []

      current_page = page_num

      if page_num == 1
        output_path = File.join(dist_dir, 'index.html')
      else
        output_dir = File.join(dist_dir, "page#{page_num}")
        FileUtils.mkdir_p(output_dir)
        output_path = File.join(output_dir, 'index.html')
      end

      html = template.result(binding)
      File.write(output_path, html)
      puts "Generated #{output_path} (#{articles.length} articles)"
    end
  end

  desc "Generate 404.html from template"
  task :generate_404 do
    template_file = File.expand_path('templates/404.html', __dir__)
    output_file = File.expand_path('dist/404.html', __dir__)

    unless File.exist?(template_file)
      puts "Warning: Template file #{template_file} not found."
      next
    end

    FileUtils.cp(template_file, output_file)
    puts "Generated #{output_file}"
  end

  desc "Generate static pages from misc directory"
  task :generate_pages do
    config = load_config
    dist_dir = File.expand_path('dist', __dir__)
    misc_dir = File.expand_path('misc', __dir__)
    template_file = File.expand_path('templates/page.html.erb', __dir__)

    unless File.exist?(template_file)
      puts "Warning: Template file #{template_file} not found. Skipping page generation."
      next
    end

    unless File.directory?(misc_dir)
      puts "Warning: misc directory not found. Skipping page generation."
      next
    end

    template = ERB.new(File.read(template_file))

    Dir.glob(File.join(misc_dir, "*.md")).each do |md_file_path|
      content = File.read(md_file_path)
      front_matter, markdown_content = parse_front_matter(content)

      page_name = File.basename(md_file_path, ".md")
      page_title = front_matter['title'] || page_name.gsub('_', ' ').capitalize
      page_content = markdown_to_html(markdown_content)

      page_dir = File.join(dist_dir, page_name)
      FileUtils.mkdir_p(page_dir)

      html = template.result(binding)
      File.write(File.join(page_dir, 'index.html'), html)
      puts "Generated #{page_dir}/index.html"
    end
  end

  desc "Build Prism.js bundle"
  task :build_prism do
    puts "Building Prism.js bundle..."
    system("npm run build:prism")
  end

  desc "Build CSS with Tailwind"
  task :build_css do
    puts "Building CSS with Tailwind..."
    system("npm run build:css")
  end

  desc "Clean generated files"
  task :clean do
    dist_dir = File.expand_path('dist', __dir__)

    # Remove generated article directories (yyyy/mm/dd/slug)
    Dir.glob(File.join(dist_dir, '[0-9][0-9][0-9][0-9]')).each do |year_dir|
      FileUtils.rm_rf(year_dir)
      puts "Removed #{year_dir}"
    end

    # Remove page directories
    Dir.glob(File.join(dist_dir, 'page[0-9]*')).each do |page_dir|
      FileUtils.rm_rf(page_dir)
      puts "Removed #{page_dir}"
    end

    # Remove generated directory
    generated_dir = File.join(dist_dir, 'generated')
    if File.directory?(generated_dir)
      FileUtils.rm_rf(generated_dir)
      puts "Removed #{generated_dir}"
    end

    puts "Clean complete!"
  end

  desc "Build site for deployment"
  task :build do
    dist_dir = File.expand_path('dist', __dir__)
    FileUtils.mkdir_p(dist_dir)

    puts "Building static blog..."

    # Build Prism.js bundle
    Rake::Task['buddie:build_prism'].invoke

    # Build CSS
    Rake::Task['buddie:build_css'].invoke

    # Generate 404.html from template
    Rake::Task['buddie:generate_404'].invoke

    # Generate article HTML pages
    Rake::Task['buddie:generate_articles'].invoke

    # Generate index pages with pagination
    Rake::Task['buddie:generate_index_pages'].invoke

    # Generate static pages from misc
    Rake::Task['buddie:generate_pages'].invoke

    puts "Build complete! Output in #{dist_dir}"
  end
end
