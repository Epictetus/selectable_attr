# SelectableAttr [![Build Status](https://secure.travis-ci.org/akm/selectable_attr.png)](http://travis-ci.org/akm/selectable_attr)

## Introduction
selectable_attr は、コードが割り振られるような特定の属性について*コード*、*プログラム上での名前*、
*表示するための名前*などをまとめて管理するものです。
http://github.com/akm/selectable_attr/tree/master

Railsで使用する場合、selectable_attr_railsと一緒に使うことをオススメします。
http://github.com/akm/selectable_attr_rails/tree/master


## Install
### 1. Railsプロジェクトで使う場合
#### a. plugin install
    ruby script/plugin install git://github.com/akm/selectable_attr.git
    ruby script/plugin install git://github.com/akm/selectable_attr_rails.git

#### b. gem install
    [sudo] gem install akimatter-selectable_attr akimatter-selectable_attr_rails -s http://gems.github .com

### 2. 非Railsで使う場合
#### a. gem install
    [sudo] gem install akimatter-selectable_attr -s http://gems.github .com



## Tutorial

### シンプルなパターン
    require 'rubygems'
    require 'selectable_attr'

    class Person
      include ::SelectableAttr::Base

      selectable_attr :gender do
        entry '1', :male, '男性'
        entry '2', :female, '女性'
        entry '9', :other, 'その他'
      end
    end

上記のようなクラスがあった場合、以下のようなクラスメソッドを使うことが可能です。

    irb(main):020:0> Person.gender_ids
    => ["1", "2", "9"]
    irb(main):021:0> Person.gender_keys
    => [:male, :female, :other]
    irb(main):022:0> Person.gender_names
    => ["男性", "女性", "その他"]
    irb(main):023:0> Person.gender_options
    => [["男性", "1"], ["女性", "2"], ["その他", "9"]] # railsでoptions_for_selectメソッドなどに使えます。
    irb(main):024:0> Person.gender_key_by_id("1")
    => :male
    irb(main):025:0> Person.gender_name_by_id("1")
    => "男性"
    irb(main):026:0> Person.gender_id_by_key(:male) # 特定のキーから対応するidを取得できます。
    => "1"
    irb(main):027:0> Person.gender_name_by_key(:male)
    => "男性"

また使用可能なインスタンスメソッドには以下のようなものがあります。
 
    irb> person = Person.new
    => #<Person:0x133b9d0>
    irb> person.gender_key
    => nil
    irb> person.gender_name
    => nil
    irb> person.gender = "2"
    => "2"
    irb> person.gender_key
    => :female
    irb> person.gender_name
    => "女性"
    irb> person.gender_key = :other
    => :other
    irb> person.gender
    => "9"
    irb> person.gender_name
    => "その他"

genderに加えて、gender_keyも代入可能となりますが、gender_nameには代入できません。

### 複数の値を取りうるパターン
    require 'rubygems'
    require 'selectable_attr'

    class RoomSearch
      include ::SelectableAttr::Base

      multi_selectable_attr :room_type do
        entry '01', :single, 'シングル'
        entry '02', :twin, 'ツイン'
        entry '03', :double, 'ダブル'
        entry '04', :triple, 'トリプル'
      end
    end

multi_selectable_attrを使った場合に使用できるクラスメソッドは、selectable_attrの場合と同じです。

    irb> room_search = RoomSearch.new
    => #<RoomSearch:0x134a070>
    irb> room_search.room_type_ids
    => []
    irb> room_search.room_type_keys
    => []
    irb> room_search.room_type_names
    => []
    irb> room_search.room_type_selection
    => [false, false, false, false]
    irb> room_search.room_type_keys = [:twin, :double]
    => [:twin, :double]
    irb> room_search.room_type
    => ["02", "03"]
    irb> room_search.room_type_names
    => ["ツイン", "ダブル"]
    irb> room_search.room_type_ids
    => ["02", "03"]
    irb> room_search.room_type = ["01", "04"]
    => ["01", "04"]
    irb> room_search.room_type_keys
    => [:single, :triple]
    irb> room_search.room_type_names
    => ["シングル", "トリプル"]
    irb> room_search.room_type_selection
    => [true, false, false, true]
    irb> room_search.room_type_hash_array
    => [{:select=>true, :key=>:single, :name=>"シングル", :id=>"01"}, {:select=>false, :key=>:twin, :name=>"ツイン", :id=>"02"}, {:select=>false, :key=>:double, :name=>"ダブル", :id=>"03"}, {:select=>true, :key=>:triple, :name=>"トリプル", :id=>"04"}]
    irb> room_search.room_type_hash_array_selected
    => [{:select=>true, :key=>:single, :name=>"シングル", :id=>"01"}, {:select=>true, :key=>:triple, :name=>"トリプル", :id=>"04"}]



### Entry
#### エントリの取得
エントリは様々な拡張が可能です。例えば以下のようにid、key、name以外の属性を設定することも可能です。

    require 'rubygems'
    require 'selectable_attr'

    class Site
      include ::SelectableAttr::Base

      selectable_attr :protocol do
        entry '01', :http , 'HTTP'      , :port => 80
        entry '02', :https, 'HTTPS'     , :port => 443
        entry '03', :ssh  , 'SSH'       , :port => 22
        entry '04', :svn  , 'Subversion', :port => 3690
      end
    end
 
クラスメソッドで各エントリを取得することが可能です。
    entry = Site.protocol_entry_by_key(:https)
    entry = Site.protocol_entry_by_id('02')

インスタンスメソッドでは以下のように取得できます。
    site = Site.new
    site.protocol_key = :https
    entry = site.protocol_entry

#### エントリの属性
id, key, nameもそのままメソッドとして用意されています。
    irb>  entry.id
    => "02"
    irb>  entry.key
    => :https
    irb>  entry.name
    => "HTTPS"

またオプションの属性もHashのようにアクセス可能です。
    irb> entry[:port]
    => 443

to_hashメソッドで、id, key, nameを含むHashを作成します。
    irb> entry.to_hash
    => {:key=>:https, :port=>443, :name=>"HTTPS", :id=>"02"}

matchメソッドでid,key,nameを除くオプションの属性群と一致しているかどうかを判断可能です。

    irb> entry.match?(:port => 22)
    => false
    irb> entry.match?(:port => 443)
    => true

ここではオプションの属性として:portしか設定していないので、matchに渡すHashのキーと値の組み合わせも一つだけですが、
複数ある場合にmatch?がtrueとなるためには、完全に一致している必要があります。


#### クラスメソッドでのエントリの扱い
クラスメソッドでエントリを取得する方法として、xxx_entry_by_id, xxx_entry_by_keyを紹介しましたが、
全てのエントリで構成される配列を取得するメソッドが xxx_entriesです。

    irb> entries = Site.protocol_entries
    irb> entries.length
    => 4

また、各エントリをto_hashでHashに変換した配列を xxx_hash_arrayメソッドで取得することも可能です。
    irb> Site.protocol_hash_array
    => [
     {:key=>:http, :port=>80, :name=>"HTTP", :id=>"01"}, 
     {:key=>:https, :port=>443, :name=>"HTTPS", :id=>"02"}, 
     {:key=>:ssh, :port=>22, :name=>"SSH", :id=>"03"}, 
     {:key=>:svn, :port=>3690, :name=>"Subversion", :id=>"04"} 
    ]


### Enum
あまり表にでてきませんが、エントリをまとめる役割のオブジェクトがEnumです。
これはクラスメソッドxxx_enumで取得することができます。

    irb> enum = Site.protocol_enum

Enumには以下のようなメソッドが用意されています。
    irb> enum.entries
    irb> enum.entries.map{|entry| entry[:port]}
    => [80, 443, 22, 3690]
 
EnumはEnumerableをincludeしているため、以下のように記述することも可能です。
    irb> enum.map{|entry| entry[:port]}
    => [80, 443, 22, 3690]

    irb>  enum.entry_by_id("03")
    => #<SelectableAttr::Enum::Entry:1352a54 @id="03", @key=:ssh, @name="SSH", @options={:port=>22}
    irb>  enum.entry_by_key(:ssh)
    => #<SelectableAttr::Enum::Entry:1352a54 @id="03", @key=:ssh, @name="SSH", @options={:port=>22}
    irb>  enum.entry_by_id_or_key(:ssh)
    => #<SelectableAttr::Enum::Entry:1352a54 @id="03", @key=:ssh, @name="SSH", @options={:port=>22}
    irb>  enum.entry_by_id_or_key('03')
    => #<SelectableAttr::Enum::Entry:1352a54 @id="03", @key=:ssh, @name="SSH", @options={:port=>22}
    irb>  enum.entry_by_hash(:port => 22)
    => #<SelectableAttr::Enum::Entry:1352a54 @id="03", @key=:ssh, @name="SSH", @options={:port=>22}
 
また、これらのメソッドが面倒と感じるようであれば、以下のような簡単なアクセスも可能です。
    irb>  enum['03']
    => #<SelectableAttr::Enum::Entry:1352a54 @id="03", @key=:ssh, @name="SSH", @options={:port=>22}
    irb>  enum[:ssh]
    => #<SelectableAttr::Enum::Entry:1352a54 @id="03", @key=:ssh, @name="SSH", @options={:port=>22}
    irb>  enum[:port => 22]
    => #<SelectableAttr::Enum::Entry:1352a54 @id="03", @key=:ssh, @name="SSH", @options={:port=>22}

またクラスメソッドで紹介したようなxxx_ids, xxx_keys, xxx_namesや、xxx_key_by_idなどのメソッドも用意されています。
    irb> enum.ids
    => ["01", "02", "03", "04"]
    irb> enum.keys
    => [:http, :https, :ssh, :svn]
    irb> enum.names
    => ["HTTP", "HTTPS", "SSH", "Subversion"]
    irb> enum.options
    => [["HTTP", "01"], ["HTTPS", "02"], ["SSH", "03"], ["Subversion", "04"]]
    irb> enum.key_by_id('04')
    => :svn
    irb> enum.id_by_key(:svn)
    => "04"
    irb> enum.name_by_id('04')
    => "Subversion"
    irb> enum.name_by_key(:svn)
    => "Subversion"
    irb> enum.to_hash_array
    => [
      {:key=>:http, :port=>80, :name=>"HTTP", :id=>"01"}, 
      {:key=>:https, :port=>443, :name=>"HTTPS", :id=>"02"}, 
      {:key=>:ssh, :port=>22, :name=>"SSH", :id=>"03"}, 
      {:key=>:svn, :port=>3690, :name=>"Subversion", :id=>"04"}
    ]

id, key以外でエントリを特定したい場合はfindメソッドが使えます。 
    irb>  enum.find(:port => 22)
    => #<SelectableAttr::Enum::Entry:1352a54 @id="03", @key=:ssh, @name="SSH", @options={:port=>22}
 
findメソッドにはブロックを渡すこともできます。
    irb>  enum.find{|entry| entry[:port] > 1024}
    => #<SelectableAttr::Enum::Entry:1352a04 @id="04", @key=:svn, @name="Subversion", @options={:port=>3690}


### Entryへのメソッド定義
entryメソッドにブロックを渡すとエントリのオブジェクトにメソッドを定義することが可能です。

    require 'rubygems'
    require 'selectable_attr'

    class Site
      include ::SelectableAttr::Base

      selectable_attr :protocol do
        entry '01', :http , 'HTTP', :port => 80 do
          def accept?(model)
            # httpで指定された場合はhttpsも可、という仕様
            model.url =~ /^http[s]{0,1}\:\/\//
          end
        end

        entry '02', :https, 'HTTPS', :port => 443 do
          def accept?(model)
            model.url =~ /^https\:\/\//
          end
        end

        entry '03', :ssh  , 'SSH', :port => 22 do
          def accept?(model)
            false
          end
        end

        entry '04', :svn  , 'Subversion', :port => 3690 do
          def accept?(model)
            model.url =~ /^svn\:\/\/|^svn+ssh\:\/\//
          end
        end

      end
    end

    enum = Site.protocol_enum

    class Project
      attr_accessor :url
    end
    project = Project.new
    project.url = "http://github.com/akm/selectable_attr/tree/master"

    irb>  enum[:http].accept?(project)
    => 0
    irb>  enum[:https].accept?(project)
    => nil

 というようにentryメソッドに渡したブロックは、生成されるエントリオブジェクトのコンテキストでinstance_evalされるので、そのメソッドを定義することが可能です。


## Credit
Copyright (c) 2008 Takeshi AKIMA, released under the MIT lice nse
