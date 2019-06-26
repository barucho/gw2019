# Crud 101


ps -ef |grep mongo |grep -v grep |awk '{print "kill " $2}'
rm -rf /u01/data/mongo01
mkdir --p /u01/data/mongo01

mongod --dbpath /u01/data/mongo01 --logpath /u01/data/mongo01/log.log  --logappend  --oplogSize 10 --smallfiles --fork


mongoimport --drop -d pcat -c products /u01/demo/products.json



```js

db.products.insertMany([
{ "_id" : "ac3", "name" : "AC3 Phone", "brand" : "ACME", "type" : "phone", "price" : 200, "warranty_years" : 1, "available" : true },
{ "_id" : "ac7", "name" : "AC7 Phone", "brand" : "ACME", "type" : "phone", "price" : 320, "warranty_years" : 1, "available" : false }
])

db.products.find({})

db.products.insertOne({
    "_id" : "ac9",
    "name" : "AC9 Phone",
    "brand" : "ACME",
    "type" : "phone",
    "price" : 333,
    "warranty_years" : 0.25,
    "available" : true
})

// select * from  products where id = "ac9"
db.products.find("_id" : "ac9");

/// lets use varibales 

t = db.products

//set myobj to one josn
t = db.products
myobj = t.findOne({_id : ObjectId("507d95d5719dbef170f15c00")})

myobj.limits.voice


//change term_years to 3
myobj.term_years=3
//save
t.save(myobj)
myobj.limits.sms.over_rate=0.01
t.save(myobj)

myobj = t.findOne({_id : ObjectId("507d95d5719dbef170f15c00")})




t.find({"limits.voice":{$exists:true}})

t.find({"limits.voice":{$exists:true}}).count()

// 3


// function
productscode = { }

productscode.b = function() { 
    var x = db.products.findOne( { _id : ObjectId("507d95d5719dbef170f15c00") } );
    if( x == null ) { 
        print("hmmm, something isn't right? try again");
        return 0;
    }
    return "" + x.limits.voice.over_rate + x.limits.sms.over_rate + x.monthly_price + x.term_years + db.products.find({term_years:3}).count();
}



````

