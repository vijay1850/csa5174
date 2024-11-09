#include <stdio.h>

// Initial Permutation (IP) table
int IP[] = { 2, 6, 3, 1, 4, 8, 5, 7 };

// Inverse Initial Permutation (IP^-1) table
int IP_inv[] = { 4, 1, 3, 5, 7, 2, 8, 6 };

// Expansion (E) table
int E[] = { 4, 1, 2, 3, 2, 3, 4, 1 };

// Permutation (P) table
int P[] = { 2, 4, 3, 1 };

// S-Boxes
int S[2][4][4] = {
    { { 1, 0, 3, 2 },   // S0
      { 3, 2, 1, 0 },
      { 0, 2, 1, 3 },
      { 3, 1, 3, 2 } },

    { { 0, 1, 2, 3 },   // S1
      { 2, 0, 1, 3 },
      { 3, 0, 1, 0 },
      { 2, 1, 0, 3 } }
};

// Initial permutation (IP)
void initialPermutation(int *input, int *output) {
    for (int i = 0; i < 8; i++) {
        output[i] = input[IP[i] - 1];
    }
}

// Inverse initial permutation (IP^-1)
void inverseInitialPermutation(int *input, int *output) {
    for (int i = 0; i < 8; i++) {
        output[i] = input[IP_inv[i] - 1];
    }
}

// Expansion (E)
void expansion(int *input, int *output) {
    for (int i = 0; i < 8; i++) {
        output[i] = input[E[i] - 1];
    }
}

// Permutation (P)
void permutation(int *input, int *output) {
    for (int i = 0; i < 4; i++) {
        output[i] = input[P[i] - 1];
    }
}

// S-Box substitution
void sBox(int *input, int *output) {
    int row = input[0] * 2 + input[3];
    int col = input[1] * 2 + input[2];
    int val = S[0][row][col]; // For S0
    output[0] = (val >> 1) & 1;
    output[1] = val & 1;

    val = S[1][row][col]; // For S1
    output[2] = (val >> 1) & 1;
    output[3] = val & 1;
}

// XOR operation
void xorOperation(int *a, int *b, int len) {
    for (int i = 0; i < len; i++) {
        a[i] ^= b[i];
    }
}

// S-DES round function
void roundFunction(int *input, int *key, int *output) {
    int expanded[8], temp[8], sboxed[4], permuted[4];
    expansion(input, expanded);
    xorOperation(expanded, key, 8);
    for (int i = 0; i < 4; i++) {
        int chunk[4];
        for (int j = 0; j < 4; j++) {
            chunk[j] = expanded[i * 4 + j];
        }
        sBox(chunk, sboxed);
        for (int j = 0; j < 2; j++) {
            temp[i * 2 + j] = sboxed[j];
        }
    }
    permutation(temp, permuted);
    for (int i = 0; i < 4; i++) {
        output[i] = permuted[i];
    }
}

// Generate the next counter value
void generateCounter(int *counter) {
    for (int i = 7; i >= 0; i--) {
        if (counter[i] == 0) {
            counter[i] = 1;
            break;
        } else {
            counter[i] = 0;
        }
    }
}

// Encrypt plaintext using S-DES in counter mode
void encrypt(int *plaintext, int *key, int *ciphertext) {
    int counter[8] = {0}; // Initialize counter to all zeros
    int temp[8], roundOutput[4];
    initialPermutation(counter, temp); // Initial permutation of counter
    for (int i = 0; i < 3; i++) {
        generateCounter(counter); // Generate next counter value
        roundFunction(temp, key, roundOutput); // Apply S-DES round function
        xorOperation(roundOutput, plaintext + i * 4, 4); // XOR with plaintext
        inverseInitialPermutation(roundOutput, temp); // Inverse initial permutation
        for (int j = 0; j < 4; j++) {
            ciphertext[i * 4 + j] = temp[j];
        }
    }
}

// Decrypt ciphertext using S-DES in counter mode
void decrypt(int *ciphertext, int *key, int *plaintext) {
    int counter[8] = {0}; // Initialize counter to all zeros
    int temp[8], roundOutput[4];
    initialPermutation(counter, temp); // Initial permutation of counter
    for (int i = 0; i < 3; i++) {
        generateCounter(counter); // Generate next counter value
        roundFunction(temp, key, roundOutput); // Apply S-DES round function
        xorOperation(roundOutput, ciphertext + i * 4, 4); // XOR with ciphertext
        inverseInitialPermutation(roundOutput, temp); // Inverse initial permutation
        for (int j = 0; j < 4; j++) {
            plaintext[i * 4 + j] = temp[j];
        }
    }
}

// Convert binary array to binary string
void binaryArrayToString(int *binary, char *str) {
    for (int i = 0; i < 8; i++) {
        str[i] = binary[i] + '0';
    }
    str[8] = '\0';
}

int main() {
    int plaintext[] = {0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 1, 0, 0, 0, 1, 0};
    int key[] = {0, 1, 1, 1, 1, 1, 1, 0, 1, 1, 1, 1, 1, 0, 1, 0};
    int ciphertext[36];
    int decrypted[36];
    encrypt(plaintext, key, ciphertext);
    decrypt(ciphertext, key, decrypted);
    char binaryStr[9];

    printf("Plaintext: ");
    binaryArrayToString(plaintext, binaryStr);
    printf("%s\n", binaryStr);

    printf("Ciphertext: ");
    binaryArrayToString(ciphertext, binaryStr);
    printf("%s\n", binaryStr);

    printf("Decrypted: ");
    binaryArrayToString(decrypted, binaryStr);
    printf("%s\n", binaryStr);

    return 0;
}
