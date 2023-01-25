## Table of contents

- [Update Query with Gorm](#update-query-with-gorm)
  - [Repositories](#repositories)
  - [Handlers](#handlers)
  - [Routes](#routes)

# Update Query with Gorm

### Repositories

> File: `repositories/user.go`

- Declare `UpdateUser` interface
  ```go
  type UserRepository interface {
    FindUsers() ([]models.User, error)
    GetUser(ID int) (models.User, error)
    CreateUser(user models.User) (models.User, error)
    UpdateUser(user models.User, ID int) (models.User, error) // Write this code
  }
  ```
- Write `UpdateUser` function

  ```go
   // Write this code
  func (r *repository) UpdateUser(user models.User, ID int) (models.User, error) {
    err := r.db.Raw("UPDATE users SET name=?, email=?, password=? WHERE id=?", user.Name, user.Email, user.Password,ID).Scan(&user).Error

    return user, err
  }
  ```

### Handlers

> File: `handlers/user.go`

- Write `UpdateUser` function

  ```go
  func (h *handler) UpdateUser(c echo.Context) error {
    request := new(usersdto.UpdateUserRequest)
    if err := c.Bind(&request); err != nil {
      return c.JSON(http.StatusBadRequest, dto.ErrorResult{Code: http.StatusBadRequest, Message: err.Error()})
    }

    id, _ := strconv.Atoi(c.Param("id"))

    user, err := h.UserRepository.GetUser(id)

    if err != nil {
      return c.JSON(http.StatusBadRequest, dto.ErrorResult{Code: http.StatusBadRequest, Message: err.Error()})
    }

    if request.Name != "" {
      user.Name = request.Name
    }

    if request.Email != "" {
      user.Email = request.Email
    }

    if request.Password != "" {
      user.Password = request.Password
    }

    data, err := h.UserRepository.UpdateUser(user, id)
    if err != nil {
      return c.JSON(http.StatusInternalServerError, dto.ErrorResult{Code: http.StatusInternalServerError, Message: err.Error()})
    }

    return c.JSON(http.StatusOK, dto.SuccessResult{Code: http.StatusOK, Data: convertResponse(data)})
  }
  ```

### Routes

> File: `routes/user.go`

- Write `Update User` route with `PATCH` method

  ```go
  func UserRoutes(e *echo.Group) {
    userRepository := repositories.RepositoryUser(mysql.DB)
    h := handlers.HandlerUser(userRepository)

    e.GET("/users", h.FindUsers)
    e.GET("/user/:id", h.GetUser)
    e.POST("/user", h.CreateUser)
    e.PATCH("/user/:id", h.UpdateUser)
  }
  ```
