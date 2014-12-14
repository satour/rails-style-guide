# 序

> “ロールモデルが重要なのだ。”<br />
> -- アレックス・マーフィー巡査 / 映画『ロボコップ』の登場人物

このガイドの目的は、Ruby on Rails 4 開発における コーディングスタイルのベストプラクティスを提供することです。すでに有志によって作成された[Ruby coding style guide](https://github.com/bbatsov/ruby-style-guide)の補足資料にあたります。

記載内容の一部は Rails 4.0 以上のみを対象としています。

PDF形式やHTML形式のコピーは[Transmuter](https://github.com/TechnoGate/transmuter)を使って作成できます。


以下の言語の翻訳が利用可能です:

* [中国語(簡体)](https://github.com/JuanitoFatas/rails-style-guide/blob/master/README-zhCN.md)
*  [中国語(繁体)](https://github.com/JuanitoFatas/rails-style-guide/blob/master/README-zhTW.md)
* [ロシア語](https://github.com/arbox/rails-style-guide/blob/master/README-ruRU.md)
* [トルコ語](https://github.com/tolgaavci/rails-style-guide/blob/master/README-trTR.md)
* [日本語](hhttps://github.com/satour/rails-style-guide/blob/master/README-jaJA.md)

# The Rails Style Guide

このガイドは、Rails開発の実務におけるコーディングスタイルのベストプラクティスを目指したものです。開発における理想論は多くありますが、このガイドはあくまで実務での利用に役立つことを目的としています。このガイドの内容は絶対ではありませんが、非常に多くのRails開発者のフィードバックや資料を基にして書かれたものです。ここで改めて、彼らへ多大な感謝を示します。

## 目次

* [Configuration](#configuration)
* [Routing](#routing)
* [Controllers](#controllers)
* [Models](#models)
* [Migrations](#migrations)
* [Views](#views)
* [Internationalization](#internationalization)
* [Assets](#assets)
* [Mailers](#mailers)
* [Bundler](#bundler)
* [Flawed Gems](#flawed-gems)
* [Managing processes](#managing-processes)

## Configuration

* <a name="config-initializers"></a>
  アプリケーション起動時の処理をカスタマイズしたい場合は、 `config/initializers` 配下にその処理を記述したコードを配置しましょう。`config/initializers` 配下のコードはアプリケーション起動時に実行されます。
<sup>[[link](#config-initializers)]</sup>

* <a name="gem-initializers"></a>
  gem の初期化に必要なコードは、gem 毎に作成しましょう。また、そのファイルの名前は gem の名前と同じにしましょう。例えば、 `carrierwave.rb`、 `active_admin.rb`などです。
<sup>[[link](#gem-initializers)]</sup>

* <a name="dev-test-prod-configs"></a>
  Railsには3つの環境（development、test、 production）がありますが、環境毎に適切な設定をしましょう。
   ( `config/environments/`配下にそれぞれの環境に対応した設定ファイルが配置してあります。)
<sup>[[link](#dev-test-prod-configs)]</sup>

  * もしプリコンパイルを行う場合は、追加するアセット名を明記しましょう。:

    ```Ruby
    # config/environments/production.rb
    # Precompile additional assets (application.js, application.css,
    #and all non-JS/CSS are already added)
    config.assets.precompile += %w( rails_admin/rails_admin.css rails_admin/rails_admin.js )
    ```

* <a name="app-config"></a>
  設定をすべての環境に適用したい場合は、  `config/application.rb` に記述しましょう.
<sup>[[link](#app-config)]</sup>

* <a name="staging-like-prod"></a>
  `production`環境と同環境の `staging` 環境を作成しておきましょう。
<sup>[[link](#staging-like-prod)]</sup>

## Routing

* <a name="member-collection-routes"></a>
もしRESTfulなresourceにアクションを追加する場合（そのようなアクションが本当に必要かはわかりませんが）、 `member` と `collection` を利用しましょう。
<sup>[[link](#member-collection-routes)]</sup>

  ```Ruby
  # 悪い例
  get 'subscriptions/:id/unsubscribe'
  resources :subscriptions

  # 良い例
  resources :subscriptions do
    get 'unsubscribe', on: :member
  end

  # 悪い例
  get 'photos/search'
  resources :photos

  # 良い例
  resources :photos do
    get 'search', on: :collection
  end
  ```

* <a name="many-member-collection-routes"></a>
  複数の`member/collection`を定義しなければいけない場合は、ブロック構文を利用しましょう。<sup>[[link](#many-member-collection-routes)]</sup>


  ```Ruby
  resources :subscriptions do
    member do
      get 'unsubscribe'
      # more routes
    end
  end

  resources :photos do
    collection do
      get 'search'
      # more routes
    end
  end
  ```

* <a name="nested-routes"></a>
  ActiveRecordのモデル間の関連を表現するには、入れ子型でルートを定義すると分かりやすいでしょう。<sup>[[link](#nested-routes)]</sup>

  ```Ruby
  class Post < ActiveRecord::Base
    has_many :comments
  end

  class Comments < ActiveRecord::Base
    belongs_to :post
  end

  # routes.rb
  resources :posts do
    resources :comments
  end
  ```

* <a name="namespaced-routes"></a>
  関連する アクション・ルートをまとめるには `namespace`を利用しましょう。<sup>[[link](#namespaced-routes)]</sup>

  ```Ruby
  namespace :admin do
    # Directs /admin/products/* to Admin::ProductsController
    # (app/controllers/admin/products_controller.rb)
    resources :products
  end
  ```

* <a name="no-wild-routes"></a>
  Railsに昔あった古いルーティング記法は絶対に利用しないようにしましょう。この記法を利用した場合、すべてのアクションへのすべてのGETリクエストを許可してしまいます。<sup>[[link](#no-wild-routes)]</sup>

  ```Ruby
  # 非常に悪い例
  match ':controller(/:action(/:id(.:format)))'
  ```

* <a name="no-match-routes"></a>
  `match` を利用しないようにしましょう。もし利用する必要がある場合は、アクション毎に `:via` オプションで`[:get, :post, :patch, :put, :delete]`を指定しましょう。 <sup>[[link](#no-match-routes)]</sup>

## Controllers

* <a name="skinny-controllers"></a>
  ビジネスロジックは controller でなく model に書きましょう。controller の役割は、view層にデータを渡すこと、またはview層からデータを受け取ることのいずれかのみです。それ以外のコードは controller に記述しないようにしましょう。<sup>[[link](#skinny-controllers)]</sup>

* <a name="one-method"></a>
  （理想論ですが）すべての controller は find と new 以外のメソッドはひとつ程度に留められるようにしましょう。<sup>[[link](#one-method)]</sup>

* <a name="shared-instance-variables"></a>
  controller と view の間でやりとりするインスタンス変数は2つまでに留めておきましょう。<sup>[[link](#shared-instance-variables)]</sup>

## Models

* <a name="model-classes"></a>
  ActiveRecord を継承しないモデルも導入しましょう。
<sup>[[link](#model-classes)]</sup>

* <a name="meaningful-model-names"></a>
  モデルの名前は、意味が通じてなおかつ短いものにしましょう。その際には省略語は利用しないようにしましょう。
<sup>[[link](#meaningful-model-names)]</sup>

* <a name="activeattr-gem"></a>
  ActiveRecordのような振る舞い（validationなど）が必要なモデルには、[ActiveAttr](https://github.com/cgriego/active_attr)
   のような gem を使いましょう.
<sup>[[link](#activeattr-gem)]</sup>

  ```Ruby
  class Message
    include ActiveAttr::Model

    attribute :name
    attribute :email
    attribute :content
    attribute :priority

    attr_accessible :name, :email, :content

    validates :name, presence: true
    validates :email, format: { with: /\A[-a-z0-9_+\.]+\@([-a-z0-9]+\.)+[a-z0-9]{2,4}\z/i }
    validates :content, length: { maximum: 500 }
  end
  ```

  詳細な例は[RailsCast on the subject](http://railscasts.com/episodes/326-activeattr)に譲ります。

### ActiveRecord

* <a name="keep-ar-defaults"></a>
  データベースの操作を自由に出来ない等、特別な理由がない限り、ActiveRecord の初期値（テーブル名、主キー等）を変更しないようにしましょう。
<sup>[[link](#keep-ar-defaults)]</sup>

  ```Ruby
  # 悪い例
  class Transaction < ActiveRecord::Base
    self.table_name = 'order'
    ...
  end
  ```

* <a name="macro-style-methods"></a>
  `has_many`や`validates`などはクラス定義の最初の方に記述しましょう。（satour注：本文と例が一致していません@原文）
<sup>[[link](#macro-style-methods)]</sup>

  ```Ruby
  class User < ActiveRecord::Base
    # 一番上にデフォルト・スコープを記述する。
    default_scope { where(active: true) }

    # 定数を記述する。
    COLORS = %w(red green blue)

    # attr関連のマクロを記述する。
    attr_accessor :formatted_date_of_birth

    attr_accessible :login, :first_name, :last_name, :email, :password

    # アソシエーションに関するマクロを記述する。
    belongs_to :country

    has_many :authentications, dependent: :destroy

    # ヴァリデーションに関するマクロを記述する。
    validates :email, presence: true
    validates :username, presence: true
    validates :username, uniqueness: { case_sensitive: false }
    validates :username, format: { with: /\A[A-Za-z][A-Za-z0-9._-]{2,19}\z/ }
    validates :password, format: { with: /\A\S{8,128}\z/, allow_nil: true}

    # コールバックを記述する。
    before_save :cook
    before_save :update_username_lower

    # 上記以外のマクロがある場合、コールバックの下に続けて記述する。

    ...
  end
  ```

* <a name="has-many-through"></a>
  なるべく`has_and_belongs_to_many`より`has_many :through`を利用しましょう。`has_many :through` を使うと model を join する際に属性やヴァリデーションを利用することが出来ます。
<sup>[[link](#has-many-through)]</sup>

  ```Ruby
  # あまり良くない例
  class User < ActiveRecord::Base
    has_and_belongs_to_many :groups
  end

  class Group < ActiveRecord::Base
    has_and_belongs_to_many :users
  end

  # 良い例
  class User < ActiveRecord::Base
    has_many :memberships
    has_many :groups, through: :memberships
  end

  class Membership < ActiveRecord::Base
    belongs_to :user
    belongs_to :group
  end

  class Group < ActiveRecord::Base
    has_many :memberships
    has_many :users, through: :memberships
  end
  ```

* <a name="read-attribute"></a>
  なるべく`read_attribute(:attribute)`より`self[:attribute]`を利用しましょう。
<sup>[[link](#read-attribute)]</sup>

  ```Ruby
  # 悪い例
  def amount
    read_attribute(:amount) * 100
  end

  # 良い例
  def amount
    self[:amount] * 100
  end
  ```

* <a name="write-attribute"></a>
  なるべく`write_attribute(:attribute, value)`より`self[:attribute] = value`を利用しましょう。
<sup>[[link](#write-attribute)]</sup>

  ```Ruby
  # 悪い例
  def amount
    write_attribute(:amount, 100)
  end

  # 良い例
  def amount
    self[:amount] = 100
  end
  ```

* <a name="sexy-validations"></a>
  ["sexy"
  validations](http://thelucid.com/2010/01/08/sexy-validation-in-edge-rails-rails-3/) を利用しましょう。
<sup>[[link](#sexy-validations)]</sup>

  ```Ruby
  # 悪い例
  validates_presence_of :email

  # 良い例
  validates :email, presence: true
  ```

* <a name="custom-validator-file"></a>
  独自のヴァリデーションが２回以上呼び出される場合、もしくは独自のヴァリデーションが正規表現を含む場合は、ファイルとして切り出しましょう。
<sup>[[link](#custom-validator-file)]</sup>

  ```Ruby
  # 悪い例
  class Person
    validates :email, format: { with: /\A([^@\s]+)@((?:[-a-z0-9]+\.)+[a-z]{2,})\z/i }
  end

  # 良い例
  class EmailValidator < ActiveModel::EachValidator
    def validate_each(record, attribute, value)
      record.errors[attribute] << (options[:message] || 'is not a valid email') unless value =~ /\A([^@\s]+)@((?:[-a-z0-9]+\.)+[a-z]{2,})\z/i
    end
  end

  class Person
    validates :email, email: true
  end
  ```

* <a name="app-validators"></a>
  独自のヴァリデーション・ファイルは`app/validators`を作成し、その配下に配置しましょう。
<sup>[[link](#app-validators)]</sup>
* <a name="custom-validators-gem"></a>
* 独自のヴァリデーションを使って複数のアプリケーションをメンテナンスしている場合、もしくはそのヴァリデーションが汎用的な内容である場合、gem 化することを検討しましょう。
<sup>[[link](#custom-validators-gem)]</sup>

* <a name="named-scopes"></a>
  名前付きスコープを取り入れましょう。
<sup>[[link](#named-scopes)]</sup>

  ```Ruby
  class User < ActiveRecord::Base
    scope :active, -> { where(active: true) }
    scope :inactive, -> { where(active: false) }

    scope :with_orders, -> { joins(:orders).select('distinct(users.id)') }
  end
  ```

* <a name="named-scope-class"></a>
  名前付きスコープがラムダ式で定義されており、かつ引数が複雑になってきた際には、スコープ名と同名の`ActiveRecord::Relation`オブジェクトを返すクラスメソッドを作成することが望ましいです。そうすることで記述を簡潔にすることができます。
<sup>[[link](#named-scope-class)]</sup>

  ```Ruby
  class User < ActiveRecord::Base
    def self.with_orders
      joins(:orders).select('distinct(users.id)')
    end
  end
  ```

* <a name="beware-update-attribute"></a>
  [`update_attribute`](http://api.rubyonrails.org/classes/ActiveRecord/Persistence.html#method-i-update_attribute) は`update_attributes`と異なり、モデルのヴァリデーション実行をスキップするので、利用する際は気をつけましょう。
<sup>[[link](#beware-update-attribute)]</sup>

* <a name="user-friendly-urls"></a>
  ユーザー・フレンドリーな URL を用意しましょう。下記に例を示します。
<sup>[[link](#user-friendly-urls)]</sup>

  * ここでは、`to_param` を上書きします。このメソッドはRailsによるURL作成に使用されます。デフォルトの実装ではレコードの`id`を`String`型で返します。

      ```Ruby
      class Person
        def to_param
          "#{id} #{name}".parameterize
        end
      end
      ```

  * `to_param`の返り値をURL-friendlyな値にするために, 文字列を`parameterize` します。`parameterize`ではオブジェクトをActiveRecordの`find`メソッドで検索できるようにする為、`id`を文字列の一番先頭に置きます。

  * `friendly_id`という gem を利用しましょう。`id`のかわりに model の他の属性を使って、読みやすいURLを生成することが出来ます。

      ```Ruby
      class Person
        extend FriendlyId
        friendly_id :name, use: :slugged
      end
      ```

  より詳細な内容は [gem documentation](https://github.com/norman/friendly_id) に譲ります。

* <a name="find-each"></a>
  ActiveRecordのオブジェクトに対して繰り返し処理を行う場合は、`find_each`を利用しましょう。（`all`などで）DBから取得したレコードセットを直接繰り返し処理させる場合、すべてのオブジェクトを即時にインスタンス化することが求められ、大量にメモリを消費します。`find_each`はインスタンス化を逐次処理にできるため、メモリ量の消費を抑制することが出来ます。
<sup>[[link](#find-each)]</sup>


  ```Ruby
  # 悪い例
  Person.all.each do |person|
    person.do_awesome_stuff
  end

  Person.where('age > 21').each do |person|
    person.party_all_night!
  end

  # 良い例
  Person.find_each do |person|
    person.do_awesome_stuff
  end

  Person.where('age > 21').find_each do |person|
    person.party_all_night!
  end
  ```

* <a name="before_destroy"></a>
  [Rails creates callbacks for dependent
  associations](https://github.com/rails/rails/issues/3458) により、
  `before_destroy`には`prepend: true`オプションを付けましょう。（satour注：リンク先の内容がタイトルから連想される内容と異なる。またリンク先のissueはcloseされている。）

  ```Ruby
  # 悪い例 (super_admin? が true でもrolesが自動的にdeleteされる。)
  has_many :roles, dependent: :destroy

  before_destroy :ensure_deletable

  def ensure_deletable
    fail "Cannot delete super admin." if super_admin?
  end

  # 良い例
  has_many :roles, dependent: :destroy

  before_destroy :ensure_deletable, prepend: true

  def ensure_deletable
    fail "Cannot delete super admin." if super_admin?
  end
  ```


## Migrations

* <a name="schema-version"></a>
  `schema.rb` (または `structure.sql`) は必ずバージョン管理しましょう。
<sup>[[link](#schema-version)]</sup>

* <a name="db-schema-load"></a>
  最新のスキーマで空のデータベースを作る際には、`rake db:migrate`でなく`rake db:schema:load`を利用しましょう。
<sup>[[link](#db-schema-load)]</sup>

* <a name="default-migration-values"></a>
  フィールドの初期値の設定処理は、アプリケーションの中に記述せず、migration ファイルに記述しましょう。
<sup>[[link](#default-migration-values)]</sup>

  ```Ruby
  # 悪い例 - アプリケーションでデフォルト値を設定している。
  def amount
    self[:amount] or 0
  end
  ```

  多くのRails開発者が、初期値の設定をRailsのアプリケーション層だけで行っています。しかし、そのやり方はデータの不整合やアプリケーションのバグを引き起こす可能性が高いです。また、重要なアプリケーションは、データベースを他のアプリケーションと共有している場合が多く、単体のRailsアプリケーションでデータ保全性を担保するのは不可能です。

* <a name="foreign-key-constraints"></a>
  外部キー制約を設定しましょう。ActiveRecord は外部キー制約の機能を提供していない為、外部キー制約の設定には
  [schema_plus](https://github.com/lomba/schema_plus) や
  [foreigner](https://github.com/matthuhiggins/foreigner) のような偉大な gem を利用しましょう。
<sup>[[link](#foreign-key-constraints)]</sup>

* <a name="change-vs-up-down"></a>
  スキーマを変更する(テーブルの追加、フィールドの追加など)migrationを書くときは、`up`や`down`でなく`change`を利用しましょう。
<sup>[[link](#change-vs-up-down)]</sup>

  ```Ruby
  # 古い書き方（古いバージョンのRailsではこのような書き方しか出来ませんでした。）
  class AddNameToPeople < ActiveRecord::Migration
    def up
      add_column :people, :name, :string
    end

    def down
      remove_column :people, :name
    end
  end

  # こちらの方が好ましい
  class AddNameToPeople < ActiveRecord::Migration
    def change
      add_column :people, :name, :string
    end
  end
  ```

* <a name="no-model-class-migrations"></a>
* migration でモデルを利用してはいけません。モデルの内容は更新されていくものであり、migration でモデルを引用していた場合、migrationの実行時に不具合を起こす可能性が高いです。
<sup>[[link](#no-model-class-migrations)]</sup>

## Views

* <a name="no-direct-model-view"></a>
  モデル層から直接、viewを呼び出さないようにしましょう。
<sup>[[link](#no-direct-model-view)]</sup>

* <a name="no-complex-view-formatting"></a>
  Viewでの表示のための複雑なフォーマット処理をviewのファイルに記述してはいけません。ヘルパーメソッドに書き出しましょう。
<sup>[[link](#no-complex-view-formatting)]</sup>

* <a name="partials"></a>
  同じコードを複数箇所に書くのは非効率です。複数のviewで同じコードを書く場合は、部分テンプレートやレイアウトにまとめましょう。
<sup>[[link](#partials)]</sup>

## Internationalization

* <a name="locale-texts"></a>
  国際化（国や土地によって文字列を翻訳する）したい文字列は、すべて`config/locales`配下のlocaleファイル（辞書ファイル）で定義しましょう。
<sup>[[link](#locale-texts)]</sup>

* <a name="translated-labels"></a>
  国際化する場合、ActiveRecordのmodelのラベルには必ず対訳が必要です。各辞書ファイルで`activerecord` スコープを使って対訳を定義しましょう。
<sup>[[link](#translated-labels)]</sup>

  ```
  en:
    activerecord:
      models:
        user: Member
      attributes:
        user:
          name: 'Full name'
  ```

  この場合、`User.model_name.human` は "Member" を返し、`User.human_attribute_name("name")` は "Full name" を返します。これらの対訳はviewでのラベル表示で使用されます。


* <a name="organize-locale-files"></a>
  ActiveRecord の model の対訳と view で使用するテキストは別々の辞書ファイルに定義しましょう。model の対訳は`model`ディレクトリを作成しその配下に辞書ファイルを作成しましょう。また view での用いる対訳は`views`ディレクトリを作成し、その配下に辞書ファイルを作成しましょう.
<sup>[[link](#organize-locale-files)]</sup>

  * 辞書ファイルを格納したディレクトリが分散する場合、 `application.rb` で辞書ファイルを格納しているディレクトリを指定する必要があります。

      ```Ruby
      # config/application.rb
      config.i18n.load_path += Dir[Rails.root.join('config', 'locales', '**', '*.{rb,yml}')]
      ```

* <a name="shared-localization"></a>
  データフォーマットや通貨表記など、どのlocaleでも共通して利用したい文字列は、`locales` 直下にファイルを作成しそこに定義しましょう。
<sup>[[link](#shared-localization)]</sup>

* <a name="short-i18n"></a>
  挙動が同じなら、利用するメソッドは名称が短い方が望ましいです。`I18n.translate`ではなく`I18n.t`を使いましょう。また、`I18n.localize`でなく、`I18n.l`を使いましょう。
<sup>[[link](#short-i18n)]</sup>

* <a name="lazy-lookup"></a>
  View では、"lazy lookup" を利用しましょう。
<sup>[[link](#lazy-lookup)]</sup>

  ```
  en:
    users:
      show:
        title: 'User details page'
  ```

  辞書ファイルでこのように訳語が定義されている場合、`app/views/users/show.html.haml`では下記の呼び出し方で`users.show.title`を呼び出すことができます。このような呼び出し方を "lazy lookup" と呼びます。

  ```Ruby
  = t '.title'
  ```

* <a name="dot-separated-keys"></a>
  訳語の呼び出しに用いるキーは、ドットを使った記法を利用しましょう。その方が`:scope`オプションを利用するより読みやすく、また辞書ファイルでの訳語の定義の階層がわかりやすいです。
<sup>[[link](#dot-separated-keys)]</sup>

  ```Ruby
  # 悪い例
  I18n.t :record_invalid, :scope => [:activerecord, :errors, :messages]

  # 良い例
  I18n.t 'activerecord.errors.messages.record_invalid'

  ```

* <a name="i18n-guides"></a>
  より詳細な内容は [Rails
  Guides](http://guides.rubyonrails.org/i18n.html) に譲ります。
<sup>[[link](#i18n-guides)]</sup>

## Assets

 [assets pipeline](http://guides.rubyonrails.org/asset_pipeline.html) を使いましょう。

* <a name="reserve-app-assets"></a>
  独自のstylesheetファイル・javascriptファイル・画像ファイルは、`app/assets`配下に配置しましょう。
<sup>[[link](#reserve-app-assets)]</sup>

* <a name="lib-assets"></a>
  開発中のアプリケーションに必ずしもフィットしていない独自ライブラリは、`lib/assets`配下に配置しましょう。
<sup>[[link](#lib-assets)]</sup>

* <a name="vendor-assets"></a>
  [jQuery](http://jquery.com/)や
  [bootstrap](http://twitter.github.com/bootstrap/)のようなサードパーティーのコードは、`vendor/assets`配下に配置しましょう。
<sup>[[link](#vendor-assets)]</sup>

* <a name="gem-assets"></a>
  可能であれば、gem 化されたアセットを利用しましょう。
  (例:
  [jquery-rails](https://github.com/rails/jquery-rails),
  [jquery-ui-rails](https://github.com/joliss/jquery-ui-rails),
  [bootstrap-sass](https://github.com/thomas-mcdonald/bootstrap-sass),
  [zurb-foundation](https://github.com/zurb/foundation)).
<sup>[[link](#gem-assets)]</sup>

## Mailers

* <a name="mailer-name"></a>
  mailer の名には `SomethingMailer`といったように`Mailer`を語尾につけ、その Mailer とどの view を結びついているか分かるようにしましょう
<sup>[[link](#mailer-name)]</sup>

* <a name="html-plain-email"></a>
  html と plain-text 両方での view templateを用意するようにしましょう。.
<sup>[[link](#html-plain-email)]</sup>

* <a name="enable-delivery-errors"></a>
  delelopment 環境では、メールの送信に失敗したらエラーが発生するように設定しておきましょう。デフォルトでは、develepment環境でのメール送信失敗はエラーとならないように設定されています。
<sup>[[link](#enable-delivery-errors)]</sup>

  ```Ruby
  # config/environments/development.rb

  config.action_mailer.raise_delivery_errors = true
  ```

* <a name="local-smtp"></a>
  development 環境ではローカルのSMTPサーバーは、[Mailcatcher](https://github.com/sj26/mailcatcher) のように利用しましょう。
<sup>[[link](#local-smtp)]</sup>

  ```Ruby
  # config/environments/development.rb

  config.action_mailer.smtp_settings = {
    address: 'localhost',
    port: 1025,
    # more settings
  }
  ```

* <a name="default-hostname"></a>
  host名を設定しましょう。
<sup>[[link](#default-hostname)]</sup>

  ```Ruby
  # config/environments/development.rb
  config.action_mailer.default_url_options = { host: "#{local_ip}:3000" }

  # config/environments/production.rb
  config.action_mailer.default_url_options = { host: 'your_site.com' }

  # in your mailer class
  default_url_options[:host] = 'your_site.com'
  ```

* <a name="url-not-path-in-email"></a>
  view で email へのリンクを記載する際には、`_path`でなく`_url`を利用しましょう。`_url` が生成するURLはホスト名を含みますが、`_path`メソッドのそれは含まない為です。
<sup>[[link](#url-not-path-in-email)]</sup>

  ```Ruby
  # 悪い例
  詳細は下記のリンクをご参照ください。
  = link_to 'こちら', course_path(@course)

  # 良い例
  詳細は下記のリンクをご参照ください。
  = link_to 'こちら', course_url(@course)
  ```

* <a name="email-addresses"></a>
  `from`と`to`は下記のような記法で記述するようにしましょう。 <sup>[[link](#email-addresses)]</sup>

  ```Ruby
  # in your mailer class
  default from: 'Your Name <info@your_site.com>'
  ```

* <a name="delivery-method-test"></a>
  email 送信のテストでは、email 送信メソッドのプロトコルを`:test`に設定しましょう。
<sup>[[link](#delivery-method-test)]</sup>

  ```Ruby
  # config/environments/test.rb

  config.action_mailer.delivery_method = :test
  ```

* <a name="delivery-method-smtp"></a>
  開発時と本番運用時には、email 送信メソッドのプロトコルは`:smtp`に設定しましょう。
<sup>[[link](#delivery-method-smtp)]</sup>

  ```Ruby
  # config/environments/development.rb, config/environments/production.rb

  config.action_mailer.delivery_method = :smtp
  ```

* <a name="inline-email-styles"></a>
  外部の css ファイルを読み込めないメールクライアントがある為、html メールを送信する場合は、すべての style を template に直接記述するようにしましょう。styleをtemplateに直接記述した場合、template のコードの保守が非常に難しいものになりますが、
  [premailer-rails](https://github.com/fphilipe/premailer-rails) や
  [roadie](https://github.com/Mange/roadie) といった gem を使用することで、style や htmlタグを適切に編集することができます。
<sup>[[link](#inline-email-styles)]</sup>

* <a name="background-email"></a>
  ページ描画と email の送信を同時に行うのはやめましょう。ページ描画の遅延によって、email 送信リクエストがタイムアウトになり、email が送信されない場合があす。[sidekiq](https://github.com/mperham/sidekiq) のような gem を利用し、email をバックグラウンドのプロセスで送信することでこの問題を回避できます。
<sup>[[link](#background-email)]</sup>

## Bundler

* <a name="dev-test-gems"></a>
  開発およびテストでしか利用しない gem は、`Gemfile`にて利用する環境（`development`や`test`）を指定しましょう。
<sup>[[link](#dev-test-gems)]</sup>

* <a name="only-good-gems"></a>
  有名で利用者の多い gem を利用しましょう。もし、無名の gem を利用しなければならない場合は、利用前にソースコードをよく確認しましょう。
<sup>[[link](#only-good-gems)]</sup>

* <a name="os-specific-gemfile-locks"></a>
  gem にはOS固有のものがあります。もし、そのようなgemを利用しており、かつプロジェクトに参加している開発者が複数のOSを利用している場合、頻繁に
  `Gemfile.lock` が書き変わってしまいます。そのような状況を避けるため、`Gemfile`では OS X 特有のgem を `darwin` とグルーピングしましょう。同様に Linux 特有の gem は`linux` とグルーピングしましょう。
<sup>[[link](#os-specific-gemfile-locks)]</sup>

  ```Ruby
  # Gemfile
  group :darwin do
    gem 'rb-fsevent'
    gem 'growl'
  end

  group :linux do
    gem 'rb-inotify'
  end
  ```

  適切な環境に適切なgemを`require`させるために、下記のコードを`config/application.rb`に追記しましょう。

  ```Ruby
  platform = RUBY_PLATFORM.match(/(linux|darwin)/)[0].to_sym
  Bundler.require(platform)
  ```

* <a name="gemfile-lock"></a>
  `Gemfile.lock`はかならずバージョン管理しましょう。これはあなた方のプロジェクトの開発者全員が`bundle install`を実行する際に、同じバージョンの gem をインストールできるよう保証するためのものです。
<sup>[[link](#gemfile-lock)]</sup>

## Flawed Gems

問題をはらんでいる、もしくは陳腐化した gem の一覧です。これらの gem は利用しないことをお勧めします。

* [rmagick](http://rmagick.rubyforge.org/)
  * この gem は莫大なメモリ量を消費してしまいます。代わりに
  [minimagick](https://github.com/minimagick/minimagick) を使いましょう。

* [autotest](http://www.zenspider.com/ZSS/Products/ZenTest/)
  * [guard](https://github.com/guard/guard) や
  [watchr](https://github.com/mynyml/watchr)の方が新しく優れているため、これらを利用しましょう。

* [rcov](https://github.com/relevance/rcov)
  * コードカバレッジツールです。Ruby 1.9 以降に対応していないため、代わりに[SimpleCov](https://github.com/colszowka/simplecov)を使いましょう。


* [therubyracer](https://github.com/cowboyd/therubyracer)
  * 莫大にメモリ量を消費するため、本番環境での利用は適当ではありません。 `node.js` を用いた方法をお勧めします。

この一覧は随時更新していきます。もし有名で問題のある gem をご存知であれば、ご一報頂けると幸いです。

## Managing processes

* <a name="foreman"></a>
  プロジェクトが外部の複数のプロセスに依存している場合、[foreman](https://github.com/ddollar/foreman) を利用しましょう。
<sup>[[link](#foreman)]</sup>

# 参考文献

この文書の他にもRailsのコーディングスタイルについて、優れた文献があります。時間があるときに目を通されてみることをお勧めします:

* [The Rails 4 Way](http://www.amazon.com/The-Rails-Addison-Wesley-Professional-Ruby/dp/0321944275)
* [Ruby on Rails Guides](http://guides.rubyonrails.org/)
* [The RSpec Book](http://pragprog.com/book/achbd/the-rspec-book)
* [The Cucumber Book](http://pragprog.com/book/hwcuc/the-cucumber-book)
* [Everyday Rails Testing with RSpec](https://leanpub.com/everydayrailsrspec)
* [Better Specs for RSpec](http://betterspecs.org)

# 貢献

このガイドに書いてあることはすべて編集可能です。 Railsのコードスタイルに興味のある全ての人と共に取り組むことが私の望みです。 究極的には、全てのRubyコミュニティにとって有益なガイドを作ることができればと思っています。
遠慮せずチケットを立てたりプルリクエストを送ったりしてください。
また、このプロジェクト(とRubocop)への金銭的な貢献は、[gittip](https://www.gittip.com/bbatsov)経由で行うことができます。

[![Support via Gittip](https://rawgithub.com/twolfson/gittip-badge/0.2.0/dist/gittip.png)](https://www.gittip.com/bbatsov)

## 貢献するには

簡単です！  [contribution guidelines](https://github.com/bbatsov/rails-style-guide/blob/master/CONTRIBUTING.md) をご覧ください！

# ライセンス

![Creative Commons License](http://i.creativecommons.org/l/by/3.0/88x31.png)この著作物は、 [Creative Commons Attribution 3.0 Unported License](http://creativecommons.org/licenses/by/3.0/deed.en_US)に従います。

# このガイドを広めてください

このガイドは有志のコミュニティによって作成されるものです。そもそも存在が世に知られないと、このガイド自体が意味のないものになってしまいます。このページをご覧になった方には、ぜひこのガイドについてTwitterでつぶやいたり、友達や同僚にシェアして、このガイドの存在を広めていただけるようお願いします。 全てのコメント・提案・意見がこのガイドをより良いものにしていきます。

親愛をこめて<br/>
[Bozhidar](https://twitter.com/bbatsov)
