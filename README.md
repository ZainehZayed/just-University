# just-University
Readme file for a part of implementation:
# Textchat
---
### Android lightweight chating application with Maching learning ML-KIT
---
## Using Tech:

* Java
* Xml
* Firebase,
* Firebase Push Notification
* Retrofit,
* Text Recognition with ML-KIT
---
#### Watch Demo : https://youtu.be/S2gbnVanKDA
---
## Features

* User can register with phone number
* User can add personal info such name, profile image
* User can see userlist who are using TextChat from his/her mobile contact
* User can see who is online
* User can send text message each other
* User can chat by group
* User can see notification if someone message
* User can send message to retrieve text from image using Gallery or Camera
---


## Feature Work

I will be  added voice recognization function and also  smart reply option by analyzing previous messages

---

## Important code

* Userlist(Firebase) with comparing phone contact:

```

Cursor=getActivity().getApplicationContext().getContentResolver().query(ContactsContract.CommonDataKinds.Phone.CONTENT_URI, null, null, null, null);

        while (cursor.moveToNext()) {
            String name = cursor.getString(cursor.getColumnIndex(ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME));
            phone = cursor.getString(cursor.getColumnIndex(ContactsContract.CommonDataKinds.Phone.NUMBER));
            if (phone.length() <= 10) {
                continue;
            }
            if (!String.valueOf(phone.charAt(0)).equals("+")) {
                phone = "+88" + phone;
            }
            phone = phone.replace(" ", "");
            phone = phone.replace("-", "");
            phone = phone.replace("(", "");
            phone = phone.replace(")", "");
            final UserObject mContact = new UserObject("", name, phone);
            contactList.add(mContact);

            final FirebaseUser firebaseUser = FirebaseAuth.getInstance().getCurrentUser();
            final DatabaseReference mUserDB = FirebaseDatabase.getInstance().getReference().child("Users");

            Query query = mUserDB.orderByChild("phone").equalTo(mContact.getPhone());
            query.addListenerForSingleValueEvent(new ValueEventListener() {

                @Override
                public void onDataChange(@NonNull DataSnapshot dataSnapshot) {
                   // mUsers.clear();
                   if (dataSnapshot.exists()) {

                            for (DataSnapshot childSnapshot : dataSnapshot.getChildren()) {
                                    User user = childSnapshot.getValue( User.class );
                                    if (!user.getId().equals( firebaseUser.getUid() )) {
                                        mUsers.add(user);
                                    }
                                }

                                userAdapter = new UserAdapter( getContext(), mUsers, false );
                                recyclerView.setAdapter( userAdapter );
                            }
                        }

                @Override
                public void onCancelled(@NonNull DatabaseError databaseError) {
                }
            });
        }
        cursor.close();

}


```

* Sending Message:

```

private void sendMessage(String sender, final String receiver, String message){
        DatabaseReference reference = FirebaseDatabase.getInstance().getReference();
        HashMap<String, Object> hashMap = new HashMap<>();
        hashMap.put("sender", sender);
        hashMap.put("receiver", receiver);
        hashMap.put("message", message);
        hashMap.put("isseen", false);
        reference.child("Chats").push().setValue(hashMap);
        final DatabaseReference chatRef = FirebaseDatabase.getInstance().getReference("Chatlist")
                .child(fuser.getUid())
                .child(userid);
        chatRef.addListenerForSingleValueEvent(new ValueEventListener() {
            @Override
            public void onDataChange(@NonNull DataSnapshot dataSnapshot) {
                if (!dataSnapshot.exists()){
                    chatRef.child("id").setValue(userid);
                }
            }
            @Override
            public void onCancelled(@NonNull DatabaseError databaseError) {
            }
        });   
        final DatabaseReference chatRefReceiver = FirebaseDatabase.getInstance().getReference("Chatlist")
                .child(userid)
                .child(fuser.getUid());
        chatRefReceiver.child("id").setValue(fuser.getUid());
        final String msg = message;
        reference = FirebaseDatabase.getInstance().getReference("Users").child(fuser.getUid());
        reference.addValueEventListener(new ValueEventListener() {
            @Override
            public void onDataChange(@NonNull DataSnapshot dataSnapshot) {
                User user = dataSnapshot.getValue(User.class);
                if (notify) {
                    sendNotifiaction(receiver, user.getUsername(), msg);
                }
                notify = false;
            }
            @Override
            public void onCancelled(@NonNull DatabaseError databaseError) {

            }
        });
    }


```

* Receiving/Reading Message:

```

private void readMesagges(final String myid, final String userid, final String imageurl){
        mchat = new ArrayList<>();
        reference = FirebaseDatabase.getInstance().getReference("Chats");
        reference.addValueEventListener(new ValueEventListener() {
            @Override
            public void onDataChange(@NonNull DataSnapshot dataSnapshot) {
                mchat.clear();
                for (DataSnapshot snapshot : dataSnapshot.getChildren()){
                    Chat chat = snapshot.getValue(Chat.class);
                    if (chat.getReceiver().equals(myid) && chat.getSender().equals(userid) ||
                            chat.getReceiver().equals(userid) && chat.getSender().equals(myid)){
                        mchat.add(chat);
                    }
                    messageAdapter = new MessageAdapter(MessageActivity.this, mchat, imageurl);
                    recyclerView.setAdapter(messageAdapter);
                }
            }
            @Override
            public void onCancelled(@NonNull DatabaseError databaseError) {

            }
        }



```

* Message Notification:

```

private void sendNotifiaction(String receiver, final String username, final String message){
        DatabaseReference tokens = FirebaseDatabase.getInstance().getReference("Tokens");
        Query query = tokens.orderByKey().equalTo(receiver);
        query.addValueEventListener(new ValueEventListener() {
            @Override
            public void onDataChange(@NonNull DataSnapshot dataSnapshot) {
                for (DataSnapshot snapshot : dataSnapshot.getChildren()){
                    Token token = snapshot.getValue(Token.class);
            Data data = new Data(fuser.getUid(), R.mipmap.ic_launcher, username+": "+message, "New Message",
                            userid);
                    Sender sender = new Sender(data, token.getToken());
                    apiService.sendNotification(sender)
                            .enqueue(new Callback<MyResponse>() {
                                @Override
                                public void onResponse(Call<MyResponse> call, Response<MyResponse> response) {
                                    if (response.code() == 200){
                                        if (response.body().success != 1){
                                            Toast.makeText(MessageActivity.this, "Failed!", Toast.LENGTH_SHORT).show();
                                        }
                                    }
                                }
                                @Override
                                public void onFailure(Call<MyResponse> call, Throwable t) {
                                }
                            });
                }
            }
            @Override
            public void onCancelled(@NonNull DatabaseError databaseError) {
            }
        });

```



[Back To The Top](#Textchat)
---

Userlist(Firebase) with comparing phone contact:

Cursor=getActivity().getApplicationContext().getContentResolver().query(ContactsContract.CommonDataKinds.Phone.CONTENT_URI, null, null, null, null);

        while (cursor.moveToNext()) {
            String name = cursor.getString(cursor.getColumnIndex(ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME));
            phone = cursor.getString(cursor.getColumnIndex(ContactsContract.CommonDataKinds.Phone.NUMBER));
            if (phone.length() <= 10) {
                continue;
            }
            if (!String.valueOf(phone.charAt(0)).equals("+")) {
                phone = "+88" + phone;
            }
            phone = phone.replace(" ", "");
            phone = phone.replace("-", "");
            phone = phone.replace("(", "");
            phone = phone.replace(")", "");
            final UserObject mContact = new UserObject("", name, phone);
            contactList.add(mContact);

            final FirebaseUser firebaseUser = FirebaseAuth.getInstance().getCurrentUser();
            final DatabaseReference mUserDB = FirebaseDatabase.getInstance().getReference().child("Users");

            Query query = mUserDB.orderByChild("phone").equalTo(mContact.getPhone());
            query.addListenerForSingleValueEvent(new ValueEventListener() {

                @Override
                public void onDataChange(@NonNull DataSnapshot dataSnapshot) {
                   // mUsers.clear();
                   if (dataSnapshot.exists()) {

                            for (DataSnapshot childSnapshot : dataSnapshot.getChildren()) {
                                    User user = childSnapshot.getValue( User.class );
                                    if (!user.getId().equals( firebaseUser.getUid() )) {
                                        mUsers.add(user);
                                    }
                                }

                                userAdapter = new UserAdapter( getContext(), mUsers, false );
                                recyclerView.setAdapter( userAdapter );
                            }
                        }

                @Override
                public void onCancelled(@NonNull DatabaseError databaseError) {
                }
            });
        }
        cursor.close();

}
