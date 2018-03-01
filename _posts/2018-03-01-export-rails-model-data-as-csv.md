---
layout: post
title: Export Rails model data as CSV
---

I needed to allow my customer to export a bunch of tables. After some digging, I found a pretty neat solution.

Let's say we have a `Customer`, `Order` and `OrderItem` model. We want to allow the user to download exported data as CSV in a single zip file. To generate CSV from the model we can use following code.

{% highlight ruby %}
class Customer < ApplicationRecord

  def self.to_csv
    attributes = %w{id name email address}

    CSV.generate(headers: true) do |csv|
      csv << attributes

      all.each do |treatment|
        csv << attributes.map{ |attr| treatment.send(attr) }
      end
    end
  end
end
{% endhighlight %}

Then we need only to package up the files in `zip` file and send it to the user.

We will use a gem called `rubyzip`. To instal it add this to your `Gemfile`:

{% highlight ruby %}
gem 'rubyzip', require: 'zip'
{% endhighlight %}

We can write text files directly to an archived file using `puts`, so all we need to do is to use `Tempfile` as our backing file.

{% highlight ruby %}
module Admin
  class ExportsController < Admin::ApplicationController
    def download_export

      tables = [
        Customer,
        Order,
        OrderItem,
      ]

      filename = "shop-export-#{Date.today}.zip"
      temp_file = Tempfile.new(filename)

      begin
        ::Zip::OutputStream.open(temp_file) do |zos|
          tables.each do |table|
            zos.put_next_entry "#{table.name.downcase}.csv"
            zos.puts table.to_csv
          end
        end

        zip_data = File.read(temp_file.path)
        send_data(zip_data, type: 'application/zip', filename: filename)
      ensure
        temp_file.close
        temp_file.unlink
      end
    end
  end
end
{% endhighlight %}
