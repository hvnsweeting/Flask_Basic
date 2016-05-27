# Flask_Basic

* Step 1: Database Schema
 - Đầu tiên ta sẽ tạo 1 database schema, ở đây tôi dùng SQLite vì nó rất đơn
 giản. Chỉ cần đưa đoạn code sau đây vào file, giả sử schema.sql:

         drop table if exists entries;

         create table entries (

            id integer primary key autoincrement,

            title text not null,

            text text not null

         );

 - Ý nghĩa của đoạn code trên là nó sẽ kiểm tra, nếu chưa tồn tại table
 entries thì nó sẽ tạo table. Schema là 1 table gồm 3 column, đầu tiên là id,
  nó sẽ tự động sinh ra id cho mỗi lần insert, thứ 2, 3 là title và text,
  field của 2 column này là text và nó phải có giá trị (not null). Để sử dụng
   schema này tạo ra table bằng cách nào thì ta sẽ tìm hiểu trong các step
   tiếp theo.

* Step 2: Sử dụng file config
 - Ta cũng có thể cấu hình nó ngay trong file main (file dùng để run khi cần
run app). Nhưng để rõ ràng và sạch sẽ hơn ta sẽ ghi riêng nó ra 1 file
config, ở đây tôi đặt tên file là settings.py. Nó sẽ là các biến chữ hoa
được gán cho những thứ mà sẽ thay đổi khi app được clone về và run trên
các máy chủ khác và do người run code tự quy định:

                    DATABASE = '/tmp/flaskr.db'
                    DEBUG = True
                    SECRET_KEY = 'development key'
                    USERNAME = 'admin'
                    PASSWORD = 'default'

* Step 3: Cấu hình APP
 - Chúng ta sẽ tạo 1 file run app, tôi đặt tên là flaskr.py. Trong đó ta sẽ
 import những module cần thiết để sử dụng và tạo ra các applications thực tế
 từ những dữ liệu đã cấu hình ở file config settings.py

                 app = Flask(__name__)
                 app.config.from_pyfile('settings.py')
   from_pyfile() sẽ đọc các biến được cấu hình từ file được func gọi và sử
   dụng nó mỗi khi ta gọi app.config[]. Đầu tiên ta sẽ dùng nó để kết nối tới
    database với đường dẫn được định nghĩa ở file config.

               def connect_db():
                   return sqlite3.connect(app.config['DATABASE'])

   Cuối cùng, ta thêm các dòng sau để run app 1  cách độc lập, để debug=True
   để thấy được Exception khi deploy sai:

                if __name__ == '__main__':
                    app.run(debug=True)

* Step 4: Tạo Database
 - Bây giờ chúng ta sẽ tạo database từ giản đồ mà ta đã tạo từ step1, ta có
 thể tạo bằng cách dùng CLI được tích hợp từ lib SQLite3:

               $ sqlite3 path_file.db < schema.sql

    Trong ví dụ của chúng ta, ta sẽ sử dụng GUI để database sẽ được tạo ra
    khi ta run app. Trong flaskr.py, ta sẽ import func closing từ contextlib
    package. Sau đó ta định nghĩa 1 func init_db() để tạo database. Function
    này sẽ gọi connect_db() đã được định nghĩa ở step3:

                def init_db():
                    with closing(connect_db()) as db:
                        with app.open_resource('schema.sql', mode='r') as f:
                            db.cursor().executescript(f.read())
                        db.commit()
    closing() cho phép chúng ta giữ 1 kết nối mở với database. open_resource
    () method của app được sử dụng trực tiếp để mở 1 file và đọc nó, chúng ta
     sử dụng ở đây để execute 1 script trên các kết nối database.

     Khi chúng ta kết nối tới 1 database, chúng ta có được 1 kết nối (ở đây
     là db) có thể cung cấp cho chúng ta 1 cursor. Trên cursor đó có method
     để execute 1 script hoàn chỉnh. Cuối cùng chúng ta cần lưu nó lại bằng
     cách sử dụng commit()-đây là cách lưu dành cho SQLite mà chúng ta đang
     sử dụng.


