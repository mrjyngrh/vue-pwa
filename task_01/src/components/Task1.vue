<template>
    <h1>Task 01</h1>
    <h2>Image CRUD IndexedDB</h2>
    <div>
        <button @click="addData">Add Image</button>
        <button @click="getData">Get Image</button>
        <button @click="updateData">Update Image</button>
        <button @click="deleteData">Delete Image</button>
        <div v-if="imageData">
            <img :src="imageData.url" alt="Stored Image" style="max-width: 100%; max-height: 300px;" />
        </div>
        <!-- <p v-if="successMessage">{{ successMessage }}</p> -->
        <!-- <DAlert
            v-if="successMessage"
            :show="true"
            :message="successMessage"
            mode="success"
            @close="clearSuccessMessage"
        />
        <DAlert
            v-if="errorMessage"
            :show="true"
            :message="errorMessage"
            mode="error"
            @close="clearErrorMessage"
        /> -->
        <div v-show="successMessage" class="alert success" @click="clearSuccessMessage">
            <span>{{ successMessage }}</span>
        </div>
        <div v-show="errorMessage" class="alert error" @click="clearErrorMessage">
            <span>{{ errorMessage }}</span>
        </div>
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
    const errorMessage = ref('');

    const timeout = 2500;  // in milliseconds

    const imageUrl1 = 'https://images.unsplash.com/photo-1538099130811-745e64318258?q=80&w=1470&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D'
    const imageUrl2 = 'https://images.unsplash.com/photo-1619476006577-9d0b89f56bde?q=80&w=1064&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D'

    const storeImageInIndexedDb = (dbPromise, blob, imageId, imageUrl, isUpdate) => {
        dbPromise.then(db => {
            const transaction = db.transaction('images', 'readwrite');
            const store = transaction.objectStore('images');

            // const request = store.add({image:blob});

            if(isUpdate){
                const getRequest = store.get(imageId);
                getRequest.onsuccess = (event) => {
                    const existingData = event.target.result;
                    if(existingData){
                        const request = store.put({id:imageId, image:blob, timestamp: new Date().getTime(), url: imageUrl});
                        request.onsuccess = () => {
                            console.log('image successfully stored');
                            successMessage.value = 'Image successfully stored';

                            setTimeout(() => {
                                clearSuccessMessage();
                            }, timeout);
                        };

                        request.onerror = (event) => {
                            console.error('error storing image: ', event.target.error);
                            successError.value = 'Error storing image';

                            setTimeout(() => {
                                clearErrorMessage();
                            }, timeout);
                        };
                    }else{
                        errorMessage.value = 'Image with the specified ID does not exist for updating';
                        console.error('Image with the specified ID does not exist for updating');

                        setTimeout(() => {
                            clearErrorMessage();
                        }, timeout);
                    }
                };
                getRequest.onerror = (event) => {
                    console.error('Error getting image:', event.target.error);
                };
            }else{
                const request = store.put({id:imageId, image:blob, timestamp: new Date().getTime(), url: imageUrl});
            
                request.onsuccess = () => {
                    console.log('image successfully stored');
                    successMessage.value = 'Image successfully stored';

                    setTimeout(() => {
                        clearSuccessMessage();
                    }, timeout);
                };

                request.onerror = (event) => {
                    console.error('error storing image: ', event.target.error);
                    successError.value = 'Error storing image';

                    setTimeout(() => {
                        clearErrorMessage();
                    }, timeout);
                };
            }
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
                imageData.value = null;
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
                successMessage.value = 'Image successfully deleted';

                setTimeout(() => {
                    clearSuccessMessage();
                }, timeout);
            };

            request.onerror = (event) => {
                console.error('error deleting image: ', event.target.error);
           
                successError.value = 'Error deleting image';

                setTimeout(() => {
                    clearErrorMessage();
                }, timeout);
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
        .then(blob=>storeImageInIndexedDb(dbPromise,blob,storedImageId.value,imageUrl1,false))
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

        fetch(imageUrl2)
        .then(response=>response.blob())
        .then(blob=>storeImageInIndexedDb(dbPromise,blob,storedImageId.value,imageUrl2,true))
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

    const clearSuccessMessage = () => {
        successMessage.value = '';
    };

    const clearErrorMessage = () => {
        errorMessage.value = '';
    };

onMounted(() => {
  initializeDatabase();
});

</script>


<!-- <style>
@import 'tailwindcss/base';
@import 'tailwindcss/components';
@import 'tailwindcss/utilities';

/* Add any custom styles here */
</style> -->

<style>
.alert {
  position: fixed;
  bottom: 0;
  left: 0;
  width: 100%;
  padding: 1rem;
  background-color: white;
  border-top: 1px solid #ddd; /* Add a border to separate alerts from the content */
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.success {
  background-color: #d4edda;
  color: #155724;
  border: 1px solid #c3e6cb;
}

.error {
  background-color: #f8d7da;
  color: #721c24;
  border: 1px solid #f5c6cb;
}

.close-btn {
  background: none;
  border: none;
  cursor: pointer;
  font-size: 1rem;
}
</style>