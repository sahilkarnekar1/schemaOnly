# schemaOnly


const mongoose = require('mongoose');
const Schema = mongoose.Schema;

// User
const UserSchema = new Schema({
  phoneNumber: { type: String, required: true, unique: true },
  name: { type: String, required: true },
  profilePicture: { type: String }, 
  status: { type: String, default: '' },
  lastSeen: { type: Date },
});

// Chat
const ChatSchema = new Schema({
  participants: [{ type: Schema.Types.ObjectId, ref: 'User', required: true }],
  groupName: { type: String }, // For group chats
  groupPicture: { type: String }, // URL of group picture
  isGroup: { type: Boolean, default: false },
  createdAt: { type: Date, default: Date.now },
  updatedAt: { type: Date, default: Date.now },
  lastMessage: { type: Schema.Types.ObjectId, ref: 'Message' },
});

// Media
const MediaAttachmentSchema = new Schema({
  type: { type: String, enum: ['image', 'video', 'audio', 'document'], required: true },
  url: { type: String, required: true },
  size: { type: Number }, // Size of the media file in bytes
  duration: { type: Number }, 
  fileName: { type: String }, 
});

// Message
const MessageSchema = new Schema({
  chat: { type: Schema.Types.ObjectId, ref: 'Chat', required: true },
  sender: { type: Schema.Types.ObjectId, ref: 'User', required: true },
  text: { type: String },
  media: { type: MediaAttachmentSchema }, 
  sentAt: { type: Date, default: Date.now },
  editedAt: { type: Date },
  deletedAt: { type: Date },
  isEdited: { type: Boolean, default: false },
  isDeleted: { type: Boolean, default: false },
  isRead: { type: Boolean, default: false },
  isDelivered: { type: Boolean, default: false },
  isDisappearing: { type: Boolean, default: false },
  disappearingTime: { type: Number }, 
});

// Middleware to handle disappearing messages
MessageSchema.pre('save', function (next) {
  if (this.isDisappearing) {
    setTimeout(() => {
      this.isDeleted = true;
      this.deletedAt = new Date();
      this.save();
    }, this.disappearingTime * 1000);
  }
  next();
});

const User = mongoose.model('User', UserSchema);
const Chat = mongoose.model('Chat', ChatSchema);
const Message = mongoose.model('Message', MessageSchema);
const MediaAttachment = mongoose.model('MediaAttachment', MediaAttachmentSchema);

module.exports = { User, Chat, Message, MediaAttachment };
