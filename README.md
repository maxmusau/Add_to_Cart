# Add_to_Cart
Cart shows the products selected for buying.

# NOTE: 
## the names used in this guideline are from a different database and table. change the names where necessary to fit your records. 
## add the the columns not included in your table
this is the sql used to create the table
```
CREATE TABLE `products` (
  `product_id` int(50) NOT NULL,
  `product_name` varchar(200) NOT NULL,
  `product_desc` varchar(500) NOT NULL,
  `product_cost` int(11) NOT NULL,
  `product_discount` varchar(50) NOT NULL,
  `product_category` varchar(50) NOT NULL,
  `product_brand` varchar(50) NOT NULL,
  `image_url` varchar(500) NOT NULL,
  `color` text NOT NULL,
  `top_brand` text NOT NULL DEFAULT 'No',
  `top_deal` text NOT NULL DEFAULT 'No',
  `date_added` timestamp NOT NULL DEFAULT current_timestamp()
)
```


# 1. STEP 1: Create an HTML file called cart.html and paste the following code

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
      <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.1/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-              F3w7mX95PdgyTmZZMECAngseQB83DfGTowi0iMjiWaeVhAn4FJkqJByhZMI3AhiU" crossorigin="anonymous">
      <link href="../static/css/style.css" rel="stylesheet">
      <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/font-awesome/4.7.0/css/font-awesome.min.css">
    <link rel="stylesheet" href=
"https://cdn.jsdelivr.net/npm/bootstrap-icons@1.5.0/font/bootstrap-icons.css" />

</head>
<body>
 <br>
                         
 {% if 'cart_item' in session %}
           <p><a id="btnEmpty" href="{{ url_for('.empty_cart') }}" class="btn btn-danger">Empty Cart</a></p>
            <table class="table table-hover">
                <thead>
                    <tr>
                        <th>Product</th>
                        <th>Quantity</th>
                        <th class="text-center">Unit Price</th>
                        <th class="text-center">Price</th>
                        <th> </th>
                    </tr>
                </thead>
                <tbody>
<!--                set the variable quantity and product cost and item price-->
                {% for key, val in session['cart_item'].items() %}
                 {% set quantity = session['cart_item'][key]['quantity'] %}
                 {% set product_cost = session['cart_item'][key]['product_cost'] %}
                 {% set item_price = session['cart_item'][key]['total_price'] %}
                    <tr>
                        <td class="col-sm-8 col-md-6">
                        <div class="media">
                            <a class="thumbnail pull-left" href="">
                                <img class="media-object" width="150" height="150"
                                     src="../static/images/{{ session['cart_item'][key]['image_url'] }}"
                                     > </a>
                             <div class="text-muted">
                               <i> <h5>
                                        {{ session['cart_item'][key]['product_name'] }}
                                        Brand <span class="text-info">{{ session['cart_item'][key]['product_brand'] }}</span>

                                </h5></i>
                            </div>
                        </div></td>
                        <td class=" col-md-3" style="text-align: center">
                        <input type="email" class="form-control col-sm" value="{{ quantity }}">
                        </td>
                        <td class="col-sm-1 col-md-1 text-center"><strong>KSH &nbsp{{ product_cost }} </strong></td>
                        <td class="col-sm-1 col-md-1 text-center"><strong>KSH &nbsp{{ item_price }} </strong></td>
                        <td class="col-sm-1 col-md-1">
                        <a href="{{ url_for('.delete_product', code=session['cart_item'][key]['product_id']) }}" class="btn btn-danger">
                            <span class="fa fa-trash"></span> Remove
                        </a></td>
                    </tr>
                    {% endfor %}
                    <tr>
                        <td colspan="4"><h5>Total Quantity</h5></td>
                        <td class="text-right"><h5><strong>{{ session['all_total_quantity'] }}</strong></h5></td>
                    </tr>
                    <tr>
                        <td colspan="3"><h4>Total</h4></td>
                        <td colspan="2" class="text-right"><h4><strong>$ {{ session['all_total_price'] }}</strong></h4></td>
                    </tr>
                    <tr>
                        <td colspan="4">
                        <a href="/getproducts" class="btn btn-info">
                            <span class="fa fa-shopping-cart"></span> Continue Shopping
                        </a>
                        </td>
                         <td>
                         {% if 'customer_id' in session %}
                               <form action="/proceed_checkout" method="post">
                                   <input type="text" name="mpesa_code" placeholder="Enter Your MPESA Code">
                                   <input type="submit" class="btn btn-info" value="Checkout">
                               </form>
                             <br><br>

                             <a href="/logout">Logout</a>


                         {% else %}
                            <a href="/customer_checkout" class="btn btn-info">
                                 Checkout <span class="fa fa-check"></span>
                            </a>
                         {% endif %}
                        </td>
                    </tr>
                </tbody>
            </table>
            <div class="row">
                <div class="col-md-6">

                </div>

            </div>
          {% else %}
            <br>
           <div class="no-records alert alert-info">Your Cart is Empty, Please continue shopping</div>
            <br>

            <a href="/" class="btn btn-info">
                            <span class="fa fa-shopping-cart"></span> Continue Shopping
            </a>

             <a href="/account" class="btn btn-info">
                            <span class="fa fa-user"></span> My Profile
            </a>
          {% endif %}
        </div>
    </div>

</body>
</html>
```
#STEP2: paste the following form in your single.html
```
<legend>
                     
                <section>

                            <form action="/add" method="post" class="form-control col-sm-1" ><br><br><br>   <p class="text-muted">Enter Quantity</p><br>
                                 <input type="hidden" name="code" value="{{row[0]}}"/>
                                 <input type="number" name="quantity" placeholder="Qtty"  required><br><br>
                                <h5 class="text-warning">Price :&nbspKSH &nbsp{{row[3]}}</h5><br>
                                 <button type="submit" class="btn btn-danger">ADD TO CART</button>
                                <br><br>

                                <br>
                            </form>

                    <br>

                </section>
       </legend>
  ```


# 3.STEP 3: inside app.py create a route for adding items to the cart ('/add') and paste the following code

```
@app.route('/add', methods=['POST'])
def add_product_to_cart():
        _quantity = int(request.form['quantity'])
        _code = request.form['code']
        # validate the received values
        if _quantity and _code and request.method == 'POST':
            cursor = con.cursor(pymysql.cursors.DictCursor)
            cursor.execute("SELECT * FROM products WHERE product_id= %s", _code)
            row = cursor.fetchone()
            #An array is a collection of items stored at contiguous memory locations. The idea is to store multiple items of the same type together

            itemArray = {str(row['product_id']): {'product_name': row['product_name'], 'product_id': row['product_id'], 'quantity': _quantity, 'product_cost': row['product_cost'],
                              'image_url': row['image_url'], 'total_price': _quantity * row['product_cost'],
                                             'product_brand': row['product_brand']}}
            print((itemArray))


            all_total_price = 0
            all_total_quantity = 0
            session.modified = True
            #if there is an item already
            if 'cart_item' in session:
                #check if the product you are adding is already there
                print("The test cart",type(row['product_id']) )
                print("session hf", session['cart_item'])
                if str(row['product_id']) in session['cart_item']:
                    print("reached here 1")


                    for key, value in session['cart_item'].items():
                        #check if product is there
                        if str(row['product_id']) == key:
                            print("reached here 2")
                            #take the old quantity  which is in session with cart item and key quantity
                            old_quantity = session['cart_item'][key]['quantity']
                            #add it with new quantity to get the total quantity and make it a session
                            total_quantity = old_quantity + _quantity
                            session['cart_item'][key]['quantity'] = total_quantity
                            #now find the new price with the new total quantity and add it to the session
                            session['cart_item'][key]['total_price'] = total_quantity * row['product_cost']

                else:
                    print("reached here 3")
                    #a new product added in the cart.Merge the previous to have a new cart item with two products
                    session['cart_item'] = array_merge(session['cart_item'], itemArray)


                for key, value in session['cart_item'].items():
                    individual_quantity = int(session['cart_item'][key]['quantity'])
                    individual_price = float(session['cart_item'][key]['total_price'])
                    all_total_quantity = all_total_quantity + individual_quantity
                    all_total_price = all_total_price + individual_price

            else:
                #if the cart is empty you add the whole item array
                session['cart_item'] = itemArray
                all_total_quantity = all_total_quantity + _quantity
                #get total price by multiplyin the cost and the quantity
                all_total_price = all_total_price + _quantity * float(row['product_cost'])


            #add total quantity and total price to a session
            session['all_total_quantity'] = all_total_quantity
            session['all_total_price'] = all_total_price
            return redirect(url_for('.cart'))
        else:
            return 'Error while adding item to cart'
```

# 4. STEP 4: create another route for the cart and customer_checkout
```
@app.route('/cart')
def cart():
    return render_template('cart.html')
```
```    
@app.route('/customer_checkout')
def customer_checkout():
    if check_customer():
            return redirect('/cart')
    else:
        return redirect('/signin')
```
# 5.STEP 5:. Create another route to empty the cart
```
@app.route('/empty')
def empty_cart():
    try:
        if 'cart_item' in session or 'all_total_quantity' in session or 'all_total_price' in session:
            session.pop('cart_item', None)
            session.pop('all_total_quantity', None)
            session.pop('all_total_price', None)
            return redirect(url_for('.cart'))
        else:
            return redirect(url_for('.cart'))

    except Exception as e:
        print(e)
```
# 6. STEP 6: Create a delete route for deleting individual items in the cart
```
@app.route('/delete/<string:code>')
def delete_product(code):
    try:
        all_total_price = 0
        all_total_quantity = 0
        session.modified = True
        for item in session['cart_item'].items():
            if item[0] == code:
                session['cart_item'].pop(item[0], None)
                if 'cart_item' in session:
                    for key, value in session['cart_item'].items():
                        individual_quantity = int(session['cart_item'][key]['quantity'])
                        individual_price = float(session['cart_item'][key]['total_price'])
                        all_total_quantity = all_total_quantity + individual_quantity
                        all_total_price = all_total_price + individual_price
                break

        if all_total_quantity == 0:
            session.clear()
        else:
            session['all_total_quantity'] = all_total_quantity
            session['all_total_price'] = all_total_price

        # return redirect('/')
        return redirect(url_for('.cart'))
    except Exception as e:
        print(e)
```

#Function for merging arrays
we need to merge the arrays so that whenever a customer shops, the quantity,product_cost and total_price is updated
```
def array_merge( first_array , second_array ):
     if isinstance( first_array , list) and isinstance( second_array , list ):
      return first_array + second_array
     #takes the new product add to the existing and merge to have one array with two products
     elif isinstance( first_array , dict ) and isinstance( second_array , dict ):
      return dict( list( first_array.items() ) + list( second_array.items() ) )
     elif isinstance( first_array , set ) and isinstance( second_array , set ):
      return first_array.union( second_array )
     return False

```
#another function for customer_check
```
def check_customer():
    if 'email' in session:
        return True
    else:
        return False
 ```
#optional: checkout route
create a python file order_gen.py and paste the following function
```
import random
import string
def random_string_generator(size=10, chars=string.ascii_uppercase + string.digits):
    return ''.join(random.choice(chars) for _ in range(size))
```
##create the checkout route 
```
from  order_gen import random_string_generator
#checkout route
@app.route('/proceed_checkout', methods = ['POST','GET'])
def proceed_checkout():
    if check_customer():
        if 'cart_item' in  session:
            if request.method == 'POST':
                mpesa_code = request.form['mpesa_code']
                all_total_price = 0
                all_total_quantity = 0
                # Need to check database******************
                order_code = random_string_generator()
                for key, value in session['cart_item'].items():
                    individual_quantity = int(session['cart_item'][key]['quantity'])
                    individual_price = float(session['cart_item'][key]['total_price'])
                    product_id = session['cart_item'][key]['product_id']
                    product_name = session['cart_item'][key]['product_name']
                    product_cost = session['cart_item'][key]['product_cost']

                    all_total_quantity = all_total_quantity + individual_quantity
                    all_total_price = all_total_price + individual_price
                    print('Individual qqty',individual_quantity)
                    print('Individual price',individual_price)
                    print('product_id', product_id)
                    print('product name', product_name)
                    print('Total qtty', all_total_quantity)
                    print('Total price', all_total_price)
                    print("=================")
                    email = session['tel']
                    #session
                    if not email:
                        flash('Sorry, Error Occured during checkout, Try Again', 'danger')
                        return redirect('/signin')
                    elif not individual_price or not individual_quantity or not product_id or not product_name or not all_total_price or not all_total_quantity:
                        flash('Sorry, Error Occured during checkout, Try Again', 'danger')
                        return redirect('/cart')
                    else:
                        try:
                            sql = 'INSERT INTO `orders`(`product_name`, `product_qtty`, `product_cost`, `email`, `order_code`, `mpesa_confirmation`, `individual_total`, `all_total_price`) VALUES (%s,%s,%s,%s,%s,%s,%s,%s)'
                            cursor = con.cursor()
                            cursor.execute(sql, (product_name, individual_quantity, product_cost, email, order_code, mpesa_code, individual_price, all_total_price))
                            con.commit()

                        except Exception as e:
                            print(e)
                            flash('Sorry, Error occured during checkout, Please try again','danger')
                            return redirect('/cart')

                print('================')
                print('Total qtty', all_total_quantity)
                print('Total price', all_total_price)
                try:
                    sql2 = 'update orders set all_total_price = %s where order_code = %s'
                    cursor = con.cursor()
                    #it updates all the products to have the same price in the same order in the all total price column
                    cursor.execute(sql2,(all_total_price, order_code))
                    con.commit()
                    flash('Your Order is Complete, Please check your Orders in Your Profile','success')
                    session.pop('cart_item', None)
                    session.pop('all_total_quantity', None)
                    session.pop('all_total_price', None)
                    print('here')
                    return redirect(url_for('cart'))
                except:
                    flash('Sorry, Error occured during checkout, Please try again','danger')
                    session.pop('cart_item', None)
                    session.pop('all_total_quantity', None)
                    session.pop('all_total_price', None)
                    return redirect('/cart')
            else:
                return redirect('/cart')
        else:
            return redirect('/cart')
    else:
        flash('You must be logged in to Make a Purchase, Please Login', 'warning')
        return redirect('/signin')
  ```
  
