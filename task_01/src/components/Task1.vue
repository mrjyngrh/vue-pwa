<template>
    <div>
        <button @click="addData">Add Image</button>
        <button @click="getData">Get Image</button>
        <button @click="updateData">Update Image</button>
        <button @click="deleteData">Delete Image</button>
        <div v-if="imageData">
        <img :src="imageData.url" alt="Stored Image" style="max-width: 100%; max-height: 300px;" />
        </div>
        <p v-if="successMessage">{{ successMessage }}</p>
    </div>

</template>

<script setup>
    import { ref, onMounted } from 'vue';

    const dbName = "mario_db";
    const dbVersion = 3;

    // const storedImageId = 1;
    const storedImageId = ref(1);
    const imageData = ref(null);
    const successMessage = ref('');

    const imageUrl1 = 'https://images.unsplash.com/photo-1538099130811-745e64318258?q=80&w=1470&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D'
    const imageUrl2 = 'https://images.unsplash.com/photo-1619476006577-9d0b89f56bde?q=80&w=1064&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D'

    const storeImageInIndexedDb = (dbPromise, blob, imageId, imageUrl) => {
        dbPromise.then(db => {
            const transaction = db.transaction('images', 'readwrite');
            const store = transaction.objectStore('images');

            // const request = store.add({image:blob});

            const request = store.put({id:imageId, image:blob, timestamp: new Date().getTime(), url: imageUrl});
            
            request.onsuccess = () => {
                console.log('image successfully stored');
                successMessage.value = 'Image successfully stored';
            };

            request.onerror = (event) => {
                console.error('error storing image: ', event.target.error);
                successMessage.value = 'Error storing image';
            };
        });
    };

    const getImageInIndexedDb = (dbPromise, imageId) => {
        dbPromise.then(db => {
            const transaction = db.transaction('images', 'readonly');
            const store = transaction.objectStore('images');

            const request = store.get(imageId);

            request.onsuccess = (event) => {
            const imageRecord = event.target.result;

            if (imageRecord) {
                // Update the imageData variable with the retrieved data
                imageData.value = { id: imageRecord.id, url: imageRecord.url };
            } else {
                console.log('Image not found in IndexedDB with ID:', imageId);
            }
    };

    request.onerror = (event) => {
      console.error('Error getting image from IndexedDB:', event.target.error);
    };
        });
    };

    const deleteImageInIndexedDb = (dbPromise, imageId) => {
        dbPromise.then(db => {
            const transaction = db.transaction('images', 'readwrite');
            const store = transaction.objectStore('images');

            const request = store.delete(imageId);

            request.onsuccess = () => {
                console.log('image successfully deleted');
            };

            request.onerror = (event) => {
                console.error('error deleting image: ', event.target.error);
            };
        });
    };


    const initializeDatabase = () => {
        const deleteRequest = indexedDB.deleteDatabase(dbName);

        deleteRequest.onsuccess = () => {
            console.log('Database successfully deleted');
            createDatabase();
        };

        deleteRequest.onerror = (event) => {
            console.error('Error deleting database:', event.target.error);
        };
    };

    const createDatabase = () => {
        const dbPromise = new Promise((resolve, reject) => {
            const request = indexedDB.open(dbName, dbVersion);
            request.onupgradeneeded = (event) => {
                const db = event.target.result;
                db.createObjectStore('images', { keyPath: 'id' });
            };
            request.onsuccess = (event) => {
                resolve(event.target.result);
            };
            request.onerror = (event) => {
                reject(event.target.error);
            };
        });
        console.log('Database initialized');
    };



    const addData = () => {

        // console.log('add data button clicked');

        const dbPromise = new Promise((resolve, reject) => {
            const request = indexedDB.open(dbName, dbVersion);
            // request.onupgradeneeded=(event) => {
            //     const db = event.target.result;
            //     // db.createObjectStore('images', {keyPath:'id', autoIncrement:true});
            //     db.createObjectStore('images', {keyPath:'id'});
            // };
            request.onsuccess = (event) => {
                resolve(event.target.result);
            };
            request.onerror = (event) => {
                reject(event.target.error);
            };
        });

        fetch(imageUrl1)
        .then(response=>response.blob())
        .then(blob=>storeImageInIndexedDb(dbPromise,blob,storedImageId.value,imageUrl1))
        .catch(error=>console.error('error fetching or storing image: ', error));
    };

    const getData = () => {

        console.log('get data button clicked');
        
        const dbPromise = new Promise((resolve, reject) => {
            const request = indexedDB.open(dbName, dbVersion);

            request.onsuccess = (event) => {
                resolve(event.target.result);
            };
            request.onerror = (event) => {
                reject(event.target.error);
            };
        });

        getImageInIndexedDb(dbPromise, storedImageId.value);
    };

    const updateData = () => {

        console.log('update data button clicked');

        const dbPromise = new Promise((resolve, reject) => {
            const request = indexedDB.open(dbName, dbVersion);
            request.onupgradeneeded=(event) => {
                const db = event.target.result;
                // db.createObjectStore('images', {keyPath:'id', autoIncrement:true});
                db.createObjectStore('images', {keyPath:'id'});
            };
            request.onsuccess = (event) => {
                resolve(event.target.result);
            };
            request.onerror = (event) => {
                reject(event.target.error);
            };
        });

        fetch(imageUrl2)
        .then(response=>response.blob())
        .then(blob=>storeImageInIndexedDb(dbPromise,blob,storedImageId.value,imageUrl2))
        .catch(error=>console.error('error fetching or storing image: ', error));

    };

    const deleteData = () => {

        console.log('delete data button clicked');

        const dbPromise = new Promise((resolve, reject) => {
            const request = indexedDB.open(dbName, dbVersion);

            request.onsuccess = (event) => {
                resolve(event.target.result);
            };
            request.onerror = (event) => {
                reject(event.target.error);
            };
        });

        deleteImageInIndexedDb(dbPromise, storedImageId.value);
    };


onMounted(() => {
  initializeDatabase();
});

</script>