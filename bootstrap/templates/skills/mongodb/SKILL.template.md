---
name: mongodb
description: Apply MongoDB best practices when writing queries, designing schemas, or working with document databases. Use when working with MongoDB or Mongoose.
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
---

# MongoDB Best Practices

Apply these standards when working with MongoDB.

## Schema Design

### Document Structure

```javascript
// User document
{
  _id: ObjectId("..."),
  email: "john@example.com",
  name: "John Doe",
  role: "user",
  status: "active",

  // Embedded document (1:1 or 1:few)
  profile: {
    avatar: "https://...",
    bio: "...",
    preferences: {
      theme: "dark",
      notifications: true
    }
  },

  // Array of embedded documents (1:few)
  addresses: [
    {
      type: "home",
      street: "123 Main St",
      city: "Boston",
      country: "US"
    }
  ],

  // References (1:many or many:many)
  organizationId: ObjectId("..."),

  // Timestamps
  createdAt: ISODate("2024-01-15T10:30:00Z"),
  updatedAt: ISODate("2024-01-15T10:30:00Z")
}
```

### Embedding vs Referencing

| Embed When | Reference When |
|------------|----------------|
| Data is queried together | Data is queried independently |
| 1:1 or 1:few relationship | 1:many (unbounded) relationship |
| Data doesn't change often | Data changes frequently |
| Document < 16MB | Document would exceed 16MB |

### Mongoose Schema Example

```javascript
// models/user.model.js
import mongoose from 'mongoose';

const addressSchema = new mongoose.Schema({
  type: {
    type: String,
    enum: ['home', 'work', 'other'],
    required: true
  },
  street: { type: String, required: true },
  city: { type: String, required: true },
  state: String,
  country: { type: String, required: true },
  zipCode: String
}, { _id: false });

const userSchema = new mongoose.Schema({
  email: {
    type: String,
    required: true,
    unique: true,
    lowercase: true,
    trim: true,
    index: true
  },
  name: {
    type: String,
    required: true,
    trim: true,
    maxlength: 100
  },
  role: {
    type: String,
    enum: ['user', 'admin', 'moderator'],
    default: 'user'
  },
  status: {
    type: String,
    enum: ['active', 'inactive', 'suspended'],
    default: 'active',
    index: true
  },
  profile: {
    avatar: String,
    bio: { type: String, maxlength: 500 },
    preferences: {
      theme: { type: String, default: 'light' },
      notifications: { type: Boolean, default: true }
    }
  },
  addresses: [addressSchema],
  organizationId: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Organization',
    index: true
  },
  lastLoginAt: Date
}, {
  timestamps: true, // Adds createdAt and updatedAt
  toJSON: {
    transform: (doc, ret) => {
      ret.id = ret._id;
      delete ret._id;
      delete ret.__v;
      return ret;
    }
  }
});

// Indexes
userSchema.index({ email: 1 }, { unique: true });
userSchema.index({ organizationId: 1, status: 1 });
userSchema.index({ createdAt: -1 });

// Instance methods
userSchema.methods.isActive = function() {
  return this.status === 'active';
};

// Static methods
userSchema.statics.findByEmail = function(email) {
  return this.findOne({ email: email.toLowerCase() });
};

// Pre-save hook
userSchema.pre('save', function(next) {
  if (this.isModified('email')) {
    this.email = this.email.toLowerCase();
  }
  next();
});

export const User = mongoose.model('User', userSchema);
```

## Query Patterns

### Basic CRUD

```javascript
// Create
const user = await User.create({
  email: 'john@example.com',
  name: 'John Doe'
});

// Create many
const users = await User.insertMany([
  { email: 'user1@example.com', name: 'User 1' },
  { email: 'user2@example.com', name: 'User 2' }
]);

// Read
const user = await User.findById(id);
const user = await User.findOne({ email: 'john@example.com' });
const users = await User.find({ status: 'active' });

// Update
const user = await User.findByIdAndUpdate(
  id,
  { $set: { name: 'Jane Doe' } },
  { new: true, runValidators: true }
);

// Upsert
const user = await User.findOneAndUpdate(
  { email: 'john@example.com' },
  { $set: { name: 'John Doe' }, $setOnInsert: { role: 'user' } },
  { upsert: true, new: true }
);

// Delete
await User.findByIdAndDelete(id);
await User.deleteMany({ status: 'inactive' });
```

### Filtering and Pagination

```javascript
// Pagination with cursor (preferred)
const lastId = req.query.cursor;
const pageSize = 20;

const users = await User.find({
  status: 'active',
  ...(lastId && { _id: { $lt: lastId } })
})
  .sort({ _id: -1 })
  .limit(pageSize);

const nextCursor = users.length === pageSize
  ? users[users.length - 1]._id
  : null;

// Pagination with skip (simpler but slower for deep pages)
const page = parseInt(req.query.page) || 1;
const limit = parseInt(req.query.limit) || 20;
const skip = (page - 1) * limit;

const [users, total] = await Promise.all([
  User.find({ status: 'active' })
    .sort({ createdAt: -1 })
    .skip(skip)
    .limit(limit),
  User.countDocuments({ status: 'active' })
]);

// Complex filtering
const filters = {};

if (req.query.search) {
  filters.$or = [
    { name: { $regex: req.query.search, $options: 'i' } },
    { email: { $regex: req.query.search, $options: 'i' } }
  ];
}

if (req.query.role) {
  filters.role = req.query.role;
}

if (req.query.createdAfter) {
  filters.createdAt = { $gte: new Date(req.query.createdAfter) };
}

const users = await User.find(filters);
```

### Population (Joins)

```javascript
// Define relationship
const orderSchema = new mongoose.Schema({
  userId: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
  items: [{
    productId: { type: mongoose.Schema.Types.ObjectId, ref: 'Product' },
    quantity: Number,
    price: Number
  }]
});

// Populate single reference
const order = await Order.findById(id)
  .populate('userId', 'name email'); // Select specific fields

// Populate nested
const order = await Order.findById(id)
  .populate('userId', 'name email')
  .populate('items.productId', 'name price');

// Populate with conditions
const orders = await Order.find({ status: 'pending' })
  .populate({
    path: 'userId',
    match: { status: 'active' },
    select: 'name email'
  });
```

### Aggregation Pipeline

```javascript
// Basic aggregation
const stats = await Order.aggregate([
  // Stage 1: Match
  { $match: { status: 'completed' } },

  // Stage 2: Group
  {
    $group: {
      _id: '$userId',
      orderCount: { $sum: 1 },
      totalSpent: { $sum: '$totalAmount' },
      avgOrderValue: { $avg: '$totalAmount' }
    }
  },

  // Stage 3: Sort
  { $sort: { totalSpent: -1 } },

  // Stage 4: Limit
  { $limit: 10 }
]);

// Aggregation with lookup (join)
const userOrders = await User.aggregate([
  { $match: { status: 'active' } },

  {
    $lookup: {
      from: 'orders',
      localField: '_id',
      foreignField: 'userId',
      as: 'orders'
    }
  },

  {
    $addFields: {
      orderCount: { $size: '$orders' },
      totalSpent: { $sum: '$orders.totalAmount' }
    }
  },

  { $project: { orders: 0 } }, // Remove orders array

  { $sort: { totalSpent: -1 } }
]);

// Time-based aggregation
const dailyRevenue = await Order.aggregate([
  {
    $match: {
      status: 'completed',
      createdAt: { $gte: new Date('2024-01-01') }
    }
  },
  {
    $group: {
      _id: {
        year: { $year: '$createdAt' },
        month: { $month: '$createdAt' },
        day: { $dayOfMonth: '$createdAt' }
      },
      revenue: { $sum: '$totalAmount' },
      orderCount: { $sum: 1 }
    }
  },
  {
    $sort: { '_id.year': 1, '_id.month': 1, '_id.day': 1 }
  }
]);

// Faceted aggregation (multiple results in one query)
const results = await Product.aggregate([
  { $match: { status: 'active' } },
  {
    $facet: {
      products: [
        { $sort: { createdAt: -1 } },
        { $skip: 0 },
        { $limit: 20 }
      ],
      totalCount: [
        { $count: 'count' }
      ],
      categories: [
        { $group: { _id: '$category', count: { $sum: 1 } } }
      ]
    }
  }
]);
```

## Indexing Strategy

```javascript
// Single field index
userSchema.index({ email: 1 }); // Ascending
userSchema.index({ createdAt: -1 }); // Descending

// Compound index (order matters!)
orderSchema.index({ userId: 1, status: 1, createdAt: -1 });

// Unique index
userSchema.index({ email: 1 }, { unique: true });

// Sparse index (only index documents with the field)
userSchema.index({ phoneNumber: 1 }, { sparse: true });

// TTL index (auto-delete after time)
sessionSchema.index({ createdAt: 1 }, { expireAfterSeconds: 86400 });

// Text index (for full-text search)
productSchema.index({ name: 'text', description: 'text' });

// Partial index (index only matching documents)
orderSchema.index(
  { createdAt: -1 },
  { partialFilterExpression: { status: 'pending' } }
);
```

## Transactions

```javascript
// Using transactions
const session = await mongoose.startSession();

try {
  session.startTransaction();

  const order = await Order.create([{
    userId,
    items,
    totalAmount
  }], { session });

  await Product.updateMany(
    { _id: { $in: items.map(i => i.productId) } },
    { $inc: { stock: -1 } },
    { session }
  );

  await session.commitTransaction();
  return order[0];

} catch (error) {
  await session.abortTransaction();
  throw error;

} finally {
  session.endSession();
}
```

## Update Operators

```javascript
// $set - Set field value
await User.updateOne(
  { _id: id },
  { $set: { name: 'New Name', 'profile.bio': 'New bio' } }
);

// $unset - Remove field
await User.updateOne(
  { _id: id },
  { $unset: { temporaryField: '' } }
);

// $inc - Increment
await Product.updateOne(
  { _id: id },
  { $inc: { stock: -1, viewCount: 1 } }
);

// $push - Add to array
await User.updateOne(
  { _id: id },
  { $push: { addresses: newAddress } }
);

// $pull - Remove from array
await User.updateOne(
  { _id: id },
  { $pull: { addresses: { type: 'old' } } }
);

// $addToSet - Add unique to array
await User.updateOne(
  { _id: id },
  { $addToSet: { tags: 'premium' } }
);

// Array positional operator
await User.updateOne(
  { _id: id, 'addresses.type': 'home' },
  { $set: { 'addresses.$.street': 'New Street' } }
);
```

## Performance Tips

```javascript
// Use lean() for read-only queries
const users = await User.find({ status: 'active' }).lean();

// Select only needed fields
const users = await User.find({}, 'name email');

// Use explain() to analyze queries
const explanation = await User.find({ status: 'active' }).explain('executionStats');

// Avoid unbounded arrays
// BAD: Embedding unlimited items
{ comments: [/* thousands of comments */] }

// GOOD: Reference and paginate
{ commentCount: 1000 } // Store count, query comments separately

// Use bulk operations
await User.bulkWrite([
  { updateOne: { filter: { _id: id1 }, update: { $set: { status: 'active' } } } },
  { updateOne: { filter: { _id: id2 }, update: { $set: { status: 'inactive' } } } }
]);
```

## Anti-Patterns to Avoid

- **Unbounded arrays**: Can exceed 16MB document limit
- **Deep nesting**: Hard to query and update
- **Not using indexes**: Always index query fields
- **SELECT * equivalent**: Always project needed fields only
- **Ignoring explain()**: Always analyze slow queries
- **Not using transactions**: For multi-document updates that must be atomic
