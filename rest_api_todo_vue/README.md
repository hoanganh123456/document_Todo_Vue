## REST

Cách chạy thử:

1. ```git clone https://github.com/thanhdat21293/rest_api_todo_vue.git```

2. ```cd rest_api_todo_vue```

3. ```npm install```

4. Bật Postgresql lên, tạo một cơ sở dữ liệu

5. Sửa đổi cấu hình kết nối đến cơ sở dữ liệu ở config/config.json :
    * Bạn cần sửa những thông tin sau sao cho phù hợp với máy bạn:
    ```script
    "development": {
         "username": "postgres",
          "password": "",
          "database": "graph",
          "host"    : "localhost",
          "port"    : 5432,
    }
    ```
6. ```npm test```

* Các hàm demo để được viết dưới dạng Unit Test sử dụng Mocha, Chai, Chai_Promise.

* Ứng dụng sử dụng express ,pg-promise, cơ sở dữ liệu postgreSQL .

## Chức năng :
   * Ứng dụng này có chức năng tương tác với database như thêm,xóa,sửa và hiển thị dữ liệu trong database hiển thị qua font-end.
   
## Cách thực hiện :

   * Đầu tiên trong file index.js ta sẽ gọi file pgp.js để kết nối database.
   * Sau đó gọi modul express , body-parser(để tương tác với from)
   * Khi người dùng đi đến router **/api/todo-mvc/all** thì tất cả các dữ liệu trong database  có bảng todolist  sẽ được select hết ra giao diện font-end
   ````
        app.get('/api/todo-mvc/all', (req, res) => { // dùng để hứng request phía client
            db.any('SELECT * FROM todolist ORDER BY id DESC') //liệt kê ra các bản ghi có trong database
            .then((data) => {
                res.json(data)
            })
            .catch(error => {
                console.log(error)
            });
        });   
   ````
   * **Thêm dữ liệu** : 
   
       * Sau khi nhập dữ liệu vào ô input và ấn enter thì dữ liệu sẽ được thêm vào database .Sau khi thêm xong để hiện thị dữ liệu ra thì ta cần phải chạy select * from một lần nữa để hiện thị dữ liệu vừa thêm mới
       
       ````
            app.post('/api/todo-mvc/addTodo', (req, res) => {
                let title1 = req.body.title; // lấy request từ ô input khi người dùng yêu cầu 
                let completed1 = req.body.completed;
                db.task(t => {
                    return t.any("INSERT INTO todolist(title, completed) VALUES($1, $2)", [title1, completed1]) // thêm dữ liệu là 2 trường trong database title ,completed với 2 giá trí $1:title1 và $2:completed1 khi người dùng nhập vào
                        .then(() => {
                            return t.any('SELECT * FROM todolist ORDER BY id DESC');
                        })
                })
                .then((data) => {
                    res.json(data)
                })
                .catch(error => {
                    console.log(error)
                });
            });
            
       ````
   * **Sửa dữ liệu :**  Sau khi nhập dữ liệu vào ô input và ấn enter thì dữ liệu sẽ được  thay đổi trong  database .Sau khi sửa xong để hiện thị dữ liệu ra thì ta cần phải chạy select * from một lần nữa để hiện thị dữ liệu vừa sửa
   
   ````
        app.post('/api/todo-mvc/editTodo', (req, res) => {
            //req.body.todo = JSON.parse(req.body.todo)
            let id1 = req.body.todo.id; // lấy request khi người dùng nhập vào from
            let title1 = req.body.todo.title;
            let completed1 = req.body.todo.completed;
            db.task(t => {
                return t.any("UPDATE todolist SET title = $1, completed = $2 WHERE id = $3", [title1, completed1, id1])
                    .then(() => { // update tên bảng trong database với 2 giá trị cần thay đổi
                        return t.any('SELECT * FROM todolist ORDER BY id DESC');
                    })
            })
            .then((data) => {
                res.json(data)
            })
            .catch(error => {
                console.log(error)
            });
        });

   ````
   
   * **Xóa dữ liệu :** Sau khi xóa dữ liệu ở ô input thì dữ liệu sẽ được xóa trong  database .Sau khi xóa xong để hiện thị dữ liệu ra thì ta cần phải chạy select * from một lần nữa là dữ liệu vừa xóa sẽ biến mất.
   ````
   	app.post('/api/todo-mvc/removeTodo', (req, res) => {
        console.log(req.body)
        let id1 = req.body.id;
    
        db.task(t => {
            return t.any("DELETE FROM todolist WHERE id = $1", id1)
                .then(() => {
                    return t.any('SELECT * FROM todolist ORDER BY id DESC');
                })
        })
        .then((data) => {
            res.json(data)
        })
        .catch(error => {
            console.log(error)
        });
    	});
   ````
   * Thêm sửa khi ở trạng thái completed : 
       * Thêm : 
            ````
                app.post('/api/todo-mvc/completeTodo', (req, res) => {
                    let id1 = req.body.todo.id;
                    let completed1 = !req.body.todo.completed;
                    db.task(t => {
                        return t.any("UPDATE todolist SET completed = $1 WHERE id = $2", [completed1, id1])
                            .then(() => {
                                return t.any('SELECT * FROM todolist ORDER BY id DESC');
                            })
                    })
                    .then((data) => {
                        res.json(data)
                    })
                    .catch(error => {
                        console.log(error)
                    });
                });
                
            ````
       * Xóa : 
            ````
              app.post('/api/todo-mvc/removeCompleted', (req, res) => {
                  let todos = req.body.completedTodos;
                  let ids = todos.map(item => {
                      return item.id;
                  });
              
                  // console.log(ids)
                  let ids1 = ids.join(',');
                  db.task(t => {
                      return t.any(`DELETE FROM todolist WHERE id IN (${ids1}) AND completed = 'true'`)
                          .then(() => {
                              return t.any('SELECT * FROM todolist ORDER BY id DESC');
                          })
                  })
                  .then((data) => {
                      res.json(data)
                  })
                  .catch(error => {
                      console.log(error)
                  });
              });  
            ````
